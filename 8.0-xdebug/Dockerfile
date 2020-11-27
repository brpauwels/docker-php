FROM brpauwels/php:8.0

USER root

# PHP Extensions
ENV XDEBUG_VERSION 3.0.0

RUN apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
    && pickle install --no-interaction xdebug-${XDEBUG_VERSION} \
    && docker-php-ext-enable \
        xdebug \
    && runDeps="$( \
        		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
        			| tr ',' '\n' \
        			| sort -u \
        			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
        	)" \
    && apk add --no-cache $runDeps \
    && apk del --no-network .build-deps

# Configure PHP
COPY conf/php/php.ini /usr/local/etc/php/conf.d/

ENV PHP_IDE_CONFIG="serverName=docker"

USER www-data
