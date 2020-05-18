# Cloud Deploy

## GCP

### Cloud SQL

#### Setup

Enable API
```
gcloud services enable sqladmin.googleapis.com
```

Create instance:
```
gcloud sql instances create waymo-v2 --tier=db-n1-standard-2 --region=us-central1
```

Set root password:
```
gcloud sql users set-password root --host=% --instance waymo-v2 --password [PASSWORD]
```

Add user:
```
gcloud sql users create worker --instance waymo-v2 --host % --password [PASSWORD]
```

Create db:
```
gcloud sql databases create laravel --instance waymo-v2
```

#### Proxy

Install local proxy: https://cloud.google.com/sql/docs/mysql/connect-admin-proxy
```
curl -o cloud_sql_proxy https://dl.google.com/cloudsql/cloud_sql_proxy.darwin.amd64
chmod +x cloud_sql_proxy
```

Run proxy locally:
```
./cloud_sql_proxy -instances=waymo-digital-events:us-central1:waymo-v2=tcp:3306 -credential_file=creds.json -dir=/cloudsql
```

```
docker run -id --name=cloud-sql-proxy -v ~/cloudsql:/cloudsql -v $(pwd):/config gcr.io/cloudsql-docker/gce-proxy:1.17 /cloud_sql_proxy -dir=/cloudsql -instances=waymo-digital-events:us-central1:waymo-v2 -credential_file=/config/creds.json
```


```
docker run -i --name=cloud-sql-proxy -p 127.0.0.1:3306:3306 -v $(pwd):/config gcr.io/cloudsql-docker/gce-proxy:1.17 /cloud_sql_proxy -instances=waymo-digital-events:us-central1:waymo-v2 -credential_file=/config/creds.json
```

Test proxy:
```
mysql -u root -p -S /cloudsql/waymo-digital-events:us-central1:waymo-v2
```

### Docker, GCR, Build


Build locally:
```
docker build -t waymo/digital-events-platform .
```

Run locally:
```
docker run -ti -p 8080:80 waymo/digital-events-platform --name=app
```

Tag:
```
docker tag waymo/digital-events-platform gcr.io/waymo-digital-events/app
```

Push to GCR:
```
docker push gcr.io/waymo-digital-events/app
```

```
gcloud builds submit waymo/test --tag=gcr.io/waymo-digital-events/app
```

Create kubernetes cluster:
```
gcloud container clusters create waymo-cluster --num-nodes=4 --enable-autoscaling --min-nodes=1 --max-nodes=100 --enable-autorepair --scopes=service-control,service-management,compute-rw,storage-ro,cloud-platform,logging-write,monitoring-write,pubsub,datastore --machine-type=n1-standard-4 --zone=us-west1-c
```

Create kubernetes secret for db vals:
```
kubectl create secret generic cloudsql-secret --from-literal=db_user=root --from-literal=db_password=<password> --from-literal=db_name=laravel
```

Create kubernetes secret for service account:
```
kubectl create secret generic db-service-account --from-file=service_account.json=./creds.json
```


Deploy container:
```
kubectl run digital-event-app --image=gcr.io/waymo-digital-events/app --port 80
```

## Laravel

Install:
```
composer global require laravel/installer
```

Create app:
```
laravel new digital-event
```
