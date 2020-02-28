FROM php:7.4-fpm-alpine

ENV REDIS_VERSION 5.1.1
ENV MONGODB_VERSION 1.7.3
ENV AMQP_VERSION 1.9.4
ENV MEMCACHED_VERSION 3.1.5

RUN apk add --no-cache \
        make \
        openssh-client \
        git \
        nginx \
        supervisor \
        gettext \
        tzdata \
        grep \
        libmemcached \
    && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        icu-dev \
        openssl-dev \
        libxml2-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libzip-dev \
        rabbitmq-c-dev \
        gmp-dev \
        cyrus-sasl-dev \
        libmemcached-dev \
    && docker-php-ext-configure gd \
        --with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install \
        bcmath \
        gd \
        gmp \
        exif \
        intl \
        mysqli \
        opcache \
        pdo_mysql \
        soap \
        sockets \
        zip \
    && pecl update-channels \
    && pecl install redis-${REDIS_VERSION} \
    && pecl install amqp-${AMQP_VERSION} \
    && pecl install mongodb-${MONGODB_VERSION} \
    && pecl install memcached-${MEMCACHED_VERSION} \
    && docker-php-ext-enable \
        redis \
        mongodb \
        amqp \
        memcached \
    && runDeps="$( \
       	scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
       	    | tr ',' '\n' \
       	    | sort -u \
       	    | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
       	)" \
    && apk add --no-cache --virtual .php-rundeps $runDeps \
    && apk del .build-deps \
    && rm -rf /tmp/* ~/.pearrc

# Configure PHP
RUN rm -rf /usr/local/etc/php-fpm.d/zz-docker.conf
COPY conf/php/php.ini /usr/local/etc/php/conf.d/
COPY conf/php/www.conf /usr/local/etc/php-fpm.d/www.conf

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

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
    && chown -R www-data:www-data /var/tmp/nginx \
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
