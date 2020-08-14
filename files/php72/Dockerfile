ARG LNMP_PHP72_VERSION

FROM php:$LNMP_PHP72_VERSION
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/shanghai
RUN ln -snf /usr/share/zone/$TZ /etc/localtime && echo $TZ > /etc/timezone

# copy文件
COPY ./pkg/memcache.tgz /home/memcache.tgz

# 更新安装依赖包和php核心扩展
RUN apt-get update
RUN apt-get install -y git libfreetype6-dev libjpeg62-turbo-dev libpng-dev libmemcached-dev zlib1g zlib1g-dev
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/
RUN docker-php-ext-install -j$(nproc) gd zip pdo_mysql opcache mysqli bcmath
RUN apt-get install -y cron
RUN apt-get install -y supervisor

RUN pecl install redis && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini
RUN pecl install igbinary && echo "extension=igbinary.so" > /usr/local/etc/php/conf.d/igbinary.ini
RUN pecl install swoole && echo "extension=swoole.so" > /usr/local/etc/php/conf.d/swoole.ini

# RUN curl https://business.swoole.com/static/loader2.0.0/swoole_loader72.so > /usr/local/lib/php/extensions/no-debug-non-zts-20170718/swoole_loader72.so \
#     && { echo "extension=swoole_loader72.so"; \
#          echo "swoole_license_files=/data/www/xxx/license.zl"; } > /usr/local/etc/php/conf.d/swoole_loader.ini

# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH

RUN rm -r /var/lib/apt/lists/*

WORKDIR /data/www

RUN usermod -u 1000 www-data
