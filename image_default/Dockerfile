FROM php:7.4.33-fpm-alpine
WORKDIR /var/www

RUN apk add --no-cache autoconf libcurl curl-dev libjpeg jpeg-dev libpng libpng-dev icu-dev libzip libzip-dev shadow freetype freetype-dev libpq postgresql-dev
RUN usermod -u 1000 www-data && groupmod -g 1000 www-data
RUN docker-php-ext-install ctype curl gd intl mysqli pdo pdo_mysql zip exif pdo_pgsql pgsql
RUN docker-php-ext-configure gd --with-gnu-ld

# [Install redis]
RUN apk add --no-cache pcre-dev $PHPIZE_DEPS \
&& pecl install redis \
&& docker-php-ext-enable redis.so

# [Install memcache]
RUN printf "\n" | pecl install -o memcache-4.0.5.2 \
    && echo "extension=memcache.so" > /usr/local/etc/php/conf.d/memcache.ini \
    && docker-php-ext-enable memcache

# Install PHP Extensions (igbinary & memcached)
RUN apk add --no-cache --update libmemcached-libs zlib
RUN set -xe && \
    cd /tmp/ && \
    apk add --no-cache --update --virtual .phpize-deps $PHPIZE_DEPS && \
    apk add --no-cache --update --virtual .memcached-deps zlib-dev libmemcached-dev cyrus-sasl-dev && \
# Install igbinary (memcached's deps)
    pecl install igbinary && \
# Install memcached
    ( \
        pecl install --nobuild memcached && \
        cd "$(pecl config-get temp_dir)/memcached" && \
        phpize && \
        ./configure --enable-memcached-igbinary && \
        make -j$(nproc) && \
        make install && \
        cd /tmp/ \
    ) && \
# Enable PHP extensions
    docker-php-ext-enable igbinary memcached && \
    rm -rf /tmp/* && \
    apk del .memcached-deps .phpize-deps

COPY ./php.ini /usr/local/etc/php/