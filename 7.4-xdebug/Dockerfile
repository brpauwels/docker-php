FROM brpauwels/php:7.4

ENV XDEBUG_VERSION 2.9.8

USER root

RUN apk add --no-cache \
        openssh-client \
        git \
    && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
    && pecl update-channels \
    && pecl install xdebug-${XDEBUG_VERSION} \
    && docker-php-ext-enable \
        xdebug \
    && runDeps="$( \
       	scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
       	    | tr ',' '\n' \
       	    | sort -u \
       	    | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
       	)" \
    && apk add --no-cache --virtual .php-rundeps $runDeps \
    && apk del .build-deps \
    && rm -rf /tmp/* ~/.pearrc

COPY conf/php/php.ini /usr/local/etc/php/conf.d/

ENV PHP_IDE_CONFIG="serverName=docker"

USER www-data
