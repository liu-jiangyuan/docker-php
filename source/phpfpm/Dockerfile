FROM php:7.3.27-fpm-alpine3.12

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY php.ini /usr/local/etc/php

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    curl -sfL http://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer && \
    chmod +x /usr/bin/composer

#    imagemagick imagemagick-dev
RUN apk update && \
    apk add --no-cache libstdc++ freetype libpng libjpeg-turbo freetype-dev libpng-dev jpeg-dev libjpeg libjpeg-turbo-dev libwebp libwebp-dev gmp-dev icu-dev gettext gettext-dev imagemagick imagemagick-dev libzip libzip-dev git vim wget net-tools supervisor && \
    apk add --no-cache --virtual .build-deps $PHPIZE_DEPS openssl-dev pcre-dev pcre2-dev zlib-dev && \
               docker-php-ext-configure gd --with-gd --with-webp-dir --with-jpeg-dir --with-png-dir --with-zlib-dir --with-freetype-dir && \
               NUMPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
               docker-php-ext-install -j${NUMPROC} gd sockets opcache pdo_mysql pcntl mysqli gmp zip

RUN docker-php-ext-configure intl && docker-php-ext-configure gettext && \
    docker-php-ext-install intl gettext

RUN pecl install protobuf && docker-php-ext-enable protobuf
RUN pecl install imagick && docker-php-ext-enable imagick
RUN pecl install redis && docker-php-ext-enable redis
RUN pecl install mongodb && docker-php-ext-enable mongodb

RUN wget http://pecl.php.net/get/msgpack-2.0.3.tgz -O msgpack-2.0.3.tgz \
	&& mkdir -p msgpack \
	&& tar -xf msgpack-2.0.3.tgz -C msgpack --strip-components=1 \
	&& ( \
        cd msgpack \
        && phpize \
        && ./configure \
        && make \
        && make install \
    ) \
	&& rm -r msgpack \
    && docker-php-ext-enable msgpack

RUN docker-php-source extract && \
    mkdir /usr/src/php/ext/swoole && \
    curl -sfL https://github.com/swoole/swoole-src/archive/v4.5.9.tar.gz -o swoole.tar.gz && \
    tar xfz swoole.tar.gz --strip-components=1 -C /usr/src/php/ext/swoole && \
    docker-php-ext-configure swoole --enable-http2 --enable-mysqlnd --enable-openssl --enable-sockets --enable-swoole-json && \
    docker-php-ext-install -j$(nproc) swoole && \
    rm -f swoole.tar.gz $HOME/.composer/*-old.phar && \
    docker-php-source delete && \
    apk del .build-deps

RUN echo "alias ll='ls -l'" >> /etc/.bashrc && source /etc/.bashrc

WORKDIR "/var/www/html"