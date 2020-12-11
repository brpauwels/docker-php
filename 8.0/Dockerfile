FROM composer as composer

RUN git clone https://github.com/FriendsOfPHP/pickle.git \
    && cd pickle \
    && composer install --no-dev --optimize-autoloader \
    && php -d phar.readonly=0 box.phar build

FROM php:8.0-fpm-alpine

COPY --from=composer /app/pickle/pickle.phar /usr/local/bin/pickle
COPY --from=composer /usr/bin/composer /usr/local/bin/composer

# PHP Extensions
ENV AMQP_VERSION 1.10.2
ENV MEMCACHED_VERSION 3.1.5
ENV MONGODB_VERSION 1.9.0
ENV REDIS_VERSION 5.3.2

RUN apk add --no-cache \
        git \
        grep \
        make \
        nginx \
        openssh-client \
        supervisor \
    && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        # amqp
        rabbitmq-c-dev \
        # gmp
        gmp-dev \
        # intl
        icu-dev \
        # memcached
        libmemcached-dev \
        # pdo_pgsql
        postgresql-dev \
        # soap
        libxml2-dev \
        # zip
        libzip-dev \
    && docker-php-ext-install \
        bcmath \
        exif \
        gmp \
        intl \
        opcache \
        pcntl \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        soap \
        sockets \
        zip \
    # && pickle install --no-interaction amqp-${AMQP_VERSION} \
    && pickle install --no-interaction memcached-${MEMCACHED_VERSION} \
    && pickle install --no-interaction mongodb-${MONGODB_VERSION} \
    && pickle install --no-interaction redis-${REDIS_VERSION} \
    && docker-php-ext-enable \
        # amqp \
        memcached \
        mongodb \
        redis \
    && runDeps="$( \
    		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
    			| tr ',' '\n' \
    			| sort -u \
    			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    	)" \
    && apk add --no-cache $runDeps \
    && apk del --no-network .build-deps

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
