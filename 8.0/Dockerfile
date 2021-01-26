FROM php:8.0-fpm-alpine

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/
COPY --from=composer /usr/bin/composer /usr/local/bin/

RUN apk add --no-cache \
    git \
    grep \
    make \
    nginx \
    openssh-client \
    supervisor

RUN install-php-extensions \
    amqp \
    bcmath \
    exif \
    gd \
    gettext \
    gmp \
    imagick \
    intl \
    memcached \
    mongodb \
    opcache \
    pcntl \
    pdo_mysql \
    pdo_pgsql \
    redis \
    soap \
    sockets \
    tidy \
    uuid \
    zip

# Configure PHP
RUN rm -rf /usr/local/etc/php-fpm.d/zz-docker.conf
COPY conf/php/php.ini /usr/local/etc/php/conf.d/
COPY conf/php/www.conf /usr/local/etc/php-fpm.d/www.conf

# Configure Supervisor
COPY conf/supervisor/supervisord.conf /etc/supervisord.conf

# Configure Nginx
RUN rm -rf /etc/nginx/conf.d/* \
    && mkdir -p /usr/local/etc/nginx/conf.d

COPY conf/nginx/nginx.conf /etc/nginx/nginx.conf
COPY conf/nginx/default.conf /usr/local/etc/nginx/conf.d/default.conf

RUN chown -R www-data:www-data /run \
    && chown -R www-data:www-data /usr/local/etc/nginx \
    && chown -R www-data:www-data /var/lib/nginx \
    && chown -R www-data:www-data /var/log/nginx

# Setup project root
RUN mkdir -p /opt/app \
    && chown www-data:www-data /opt/app \
    && chmod 777 /opt/app

# Switch to www-data user
USER www-data

WORKDIR /opt/app

EXPOSE 8080

CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]
