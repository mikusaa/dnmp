FROM php:8.1.7-fpm-alpine

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
    && apk update \
    && apk add --virtual .build-tools --no-cache autoconf gcc g++ make \
    && apk --no-cache add ffmpeg graphicsmagick zip zlib-dev libjpeg-turbo-dev libpng-dev freetype-dev imagemagick-dev imagemagick libzip-dev icu-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd \
    && docker-php-ext-install exif mysqli pdo_mysql zip bcmath intl pcntl \
    && pecl install redis imagick \
    && docker-php-ext-enable redis imagick \
    && apk del .build-tools \
    && curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && apk add --no-cache git
