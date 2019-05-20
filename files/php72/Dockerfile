FROM php:7.2-fpm
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/shanghai
RUN ln -snf /usr/share/zone/$TZ /etc/localtime && echo $TZ > /etc/timezone

# copy文件
COPY ./pkg/memcache.tgz /home/memcache.tgz

# 更新安装依赖包和php核心扩展
RUN apt-get update && apt-get install -y git libfreetype6-dev libjpeg62-turbo-dev libpng-dev \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd zip pdo_mysql opcache mysqli \
    && rm -r /var/lib/apt/lists/*

RUN pecl install redis && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini
RUN pecl install mongodb && echo "extension=mongodb.so" > /usr/local/etc/php/conf.d/mongodb.ini
RUN pecl install igbinary && echo "extension=igbinary.so" > /usr/local/etc/php/conf.d/igbinary.ini
RUN apt-get update && apt-get install -y libmemcached-dev zlib1g zlib1g-dev && rm -r /var/lib/apt/lists/*
RUN pecl install memcached && echo "extension=memcached.so" > /usr/local/etc/php/conf.d/memcached.ini
RUN cd /home && tar -zxvf /home/memcache.tgz -C /home/ && cd /home/memcache \
    && /usr/local/bin/phpize && ./configure --with-php-config=/usr/local/bin/php-config \
    && make && make install && echo "extension=memcache.so" > /usr/local/etc/php/conf.d/memcache.ini \
    && rm -rf /home/mecache && rm -rf /home/mecache.taz
# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH

WORKDIR /data/www

RUN usermod -u 1000 www-data
