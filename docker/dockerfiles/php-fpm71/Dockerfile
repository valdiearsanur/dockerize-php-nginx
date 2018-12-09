FROM php:7.1.9-fpm
MAINTAINER Valdie <valdiearsanur@gmail.com>

ADD php.ini /usr/local/etc/php/conf.d/php.ini

RUN apt-get update \
  && docker-php-ext-install pdo_mysql mysqli mbstring

RUN apt-get update \
  && apt-get install -y git libmemcached-dev zlib1g-dev \
  && pecl install mongodb-1.4.2 \
  && pecl install memcached-3.0.3 \
  && pecl install xdebug-2.6.0 \
  && docker-php-ext-enable mongodb memcached opcache xdebug

COPY --from=composer:1.6 /usr/bin/composer /usr/bin/composer