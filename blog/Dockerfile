FROM creativitykills/nginx-php-server
COPY . /var/www/
ENV LARAVEL_APP=1
ENV COMPOSER_INSTALL_ON_BUILD=1
# WORKDIR /var/www/
RUN chmod 2775 /var/www && find /var/www -type d -exec chmod 2775 {} \;
RUN find /var/www -type f -exec chmod 0664 {} \;
RUN find /var/www/ -type d -print0 | xargs -0 chmod 755
RUN composer install
# RUN php artisan key:generate
RUN php artisan config:clear
RUN chmod -Rf 777 storage/
RUN chmod -Rf 777 bootstrap/cache
