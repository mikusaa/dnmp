FROM php:8.3.0-fpm-alpine3.18
#使用镜像加速github下载，如果服务器不在国内，可以删除 https://mirror.ghproxy.com/
ADD https://mirror.ghproxy.com/https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
    && apk update \
    && chmod +x /usr/local/bin/install-php-extensions \
    && install-php-extensions imagick bcmath pdo_mysql pdo_pgsql redis mysqli zip gd exif intl \
    && apk add --no-cache git tzdata shadow \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && usermod -u 1000 www-data \
    && groupmod -g 1000 www-data \
    && rm -rf /var/cache/apk/*

RUN curl -o /usr/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x /usr/bin/composer

ENV COMPOSER_HOME=/tmp/composer

WORKDIR /home/wwwroot