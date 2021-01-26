FROM brpauwels/php:8.0

USER root

RUN install-php-extensions xdebug

# Configure PHP
COPY conf/php/php.ini /usr/local/etc/php/conf.d/

ENV PHP_IDE_CONFIG="serverName=docker"

USER www-data
