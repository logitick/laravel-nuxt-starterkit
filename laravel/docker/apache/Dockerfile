
FROM composer:1.7 as builder

RUN mkdir -p /var/www/html
WORKDIR /var/www/html

COPY composer.json ./
COPY composer.lock ./
COPY database/ database/
RUN composer install --no-scripts --no-autoloader --no-dev --ignore-platform-reqs

COPY . /var/www/app

FROM php:7.2.27-apache-buster

ARG XDEBUG=false
ENV XDEBUG ${XDEBUG}
ARG INSTALL_COMPOSER=false
ENV INSTALL_COMPOSER ${INSTALL_COMPOSER}

## in case user has different UID
ARG PUID=1000
ARG PGID=1000
ENV PUID ${PUID}
ENV PGID ${PGID}


RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libzip-dev \
        libmagickwand-dev \
        imagemagick \
        libcurl4-openssl-dev \
    && docker-php-ext-configure gd  --with-freetype-dir=/usr --with-jpeg-dir=/usr --with-png-dir=/usr \
    && docker-php-ext-install -j$(nproc) gd \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && docker-php-ext-install zip xml intl curl\
    && docker-php-ext-install pdo pdo_mysql

# install composer
RUN if [ ${INSTALL_COMPOSER} = true ]; then \
        php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && php composer-setup.php && php -r "unlink('composer-setup.php');" && mv composer.phar /usr/bin/composer \
    ;fi

COPY . /var/www/html

WORKDIR /var/www/html

COPY --from=builder /var/www/html/vendor/ ./vendor/
RUN usermod -u ${PUID} www-data && groupmod -g ${PGID} www-data
RUN chown -R www-data:www-data /var/www && chmod -Rv ug+rwx storage bootstrap/cache

# replace default apache configuration
RUN cat docker/apache/site.conf > /etc/apache2/sites-enabled/000-default.conf

RUN a2enmod rewrite headers
CMD /usr/sbin/apache2ctl -D FOREGROUND

