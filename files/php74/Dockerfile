FROM php:7.4-fpm-alpine

ENV TIMEZONE Asia/Shanghai
ENV PHP_MEMORY_LIMIT 512M
ENV MAX_UPLOAD 50M
ENV PHP_MAX_FILE_UPLOAD 200
ENV PHP_MAX_POST 100M

# 基础依赖
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
    && apk update \
    && apk add --no-cache --update libmemcached-libs zlib libpng-dev freetype-dev libjpeg-turbo-dev \
    # 设置时区
    && ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone \
    # 安装composer
    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer \
    && composer self-update \
    # 设置配置文件
    && cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini \
    && sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /usr/local/etc/php-fpm.conf \
    && sed -i -e "s/listen\s*=\s*127.0.0.1:9000/listen = 9000/g" /usr/local/etc/php-fpm.d/www.conf \
    && sed -i "s|;date.timezone =.*|date.timezone = ${TIMEZONE}|" /usr/local/etc/php/php.ini \
    && sed -i "s|memory_limit =.*|memory_limit = ${PHP_MEMORY_LIMIT}|" /usr/local/etc/php/php.ini \
    && sed -i "s|upload_max_filesize =.*|upload_max_filesize = ${MAX_UPLOAD}|" /usr/local/etc/php/php.ini \
    && sed -i "s|max_file_uploads =.*|max_file_uploads = ${PHP_MAX_FILE_UPLOAD}|" /usr/local/etc/php/php.ini \
    && sed -i "s|post_max_size =.*|post_max_size = ${PHP_MAX_POST}|" /usr/local/etc/php/php.ini \
    && docker-php-ext-configure gd --with-jpeg=/usr/include --with-freetype=/usr/include/freetype2/ \
    && docker-php-ext-install -j$(nproc) gd bcmath opcache pdo_mysql \
    # 安装redis
    && apk add --no-cache --update --virtual .phpize-deps $PHPIZE_DEPS \
    # 防止redis报错，禁止使用pecl install redis
    && pecl install -o -f https://pecl.php.net/get/redis-5.3.7.tgz && echo "extension=redis.so" >  /usr/local/etc/php/conf.d/ext-redis.ini \
    && rm -rf /usr/share/php \
    && rm -rf /tmp/* \
    && apk del .phpize-deps \
    && rm -rf /var/cache/apk/*

WORKDIR /data/www/

EXPOSE 9000
CMD ["php-fpm"]
