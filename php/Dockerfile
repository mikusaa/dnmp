FROM php:8.3.3-fpm-alpine3.18
#如果服务器不在国内，可以在使用 https://mirror.ghproxy.com/ 镜像加速github下载，例如：
# ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

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