ARG PHP_VERSION

FROM php:${PHP_VERSION}-fpm-alpine3.16 AS dev

RUN apk update; \
    apk upgrade;

RUN apk add --update linux-headers
RUN apk add --no-cache --virtual dev-deps git autoconf gcc g++ make
RUN apk add --no-cache icu-dev jpeg-dev libpng-dev zlib-dev libzip-dev libxml2-dev

RUN docker-php-ext-configure gd --with-jpeg=/usr/lib/
RUN docker-php-ext-install mysqli pdo_mysql intl gd zip soap bcmath

RUN apk --no-cache add pcre-dev ${PHPIZE_DEPS} \ 
  && pecl install xdebug \
  && docker-php-ext-enable xdebug \
  && apk del pcre-dev ${PHPIZE_DEPS} 

RUN adduser -D -g '' stduser

RUN php -r 'readfile("https://getcomposer.org/installer");' > composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm -f composer-setup.php


COPY ./config/default.ini /usr/local/etc/php/conf.d/default.ini

ENV APP_ENV=dev

WORKDIR /var/www/html



FROM dev AS test

ENV APP_ENV=test

WORKDIR /var/www/html

## Copy project files to workdir
COPY . .

## Install application dependencies
RUN composer install --no-interaction --optimize-autoloader

## Change files owner to php-fpm default user
RUN chown -R www-data:www-data .



FROM test AS prod

ENV APP_ENV=prod

WORKDIR /var/www/html

RUN rm /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

## Cleanup
RUN apk del dev-deps && \
    composer global remove hirak/prestissimo && \
    rm /usr/local/bin/composer