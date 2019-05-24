# Docker构建自己的PHP开发环境

[TOC]
## 使用

### linux 环境
> 下面是我写的几个dockerfile启动脚本，如果自己有需要可以自己搭配环境
#### 基础的nginx+php56+php72+mysql+reids+mongo+memcached配置
```bash
$ mkdir ~/dockerapps # ln -s 自己的工作目录过来
$ cd ~/dockerapps
$ git clone https://github.com/canren/lnmp.git
$ cd lnmp
$ chmod -R 0755 ./lnmp
$ ./lnmp build # 中间断开命令重试即可
```
#### 基础的nginx+php72+mysql配置
```bash
$ mkdir ~/dockerapps # ln -s 自己的工作目录过来
$ cd ~/dockerapps
$ git clone https://github.com/canren/lnmp.git
$ cd lnmp
$ chmod -R 0755 ./lnmp72
$ ./lnmp72 build # 中间断开命令重试即可
```
#### 基础的nginx+php72+mysql配置，带nginx日志的收集的分析的脚本
>这个需要首先配置elk docker脚本,相关[https://github.com/canren/elk](https://github.com/canren/elk)，`elk脚本启动完成之后，再启动该脚本`
```bash
$ mkdir ~/dockerapps # ln -s 自己的工作目录过来
$ cd ~/dockerapps
$ git clone https://github.com/canren/lnmp.git
$ cd lnmp
$ chmod -R 0755 ./lnmp72_filebeat
$ ./lnmp72_filebeat build # 中间断开命令重试即可
```
### windows 环境
```
windows环境脚本还在编写当中
```
------------------------
### 容器相关命令

> lnmp shell
```bash
~/dockerapps/lnmp # shell路径
$ lnmp build    # 容器打包
$ lnmp start    # 开启容器
$ lnmp stop     # 关闭容器
$ lnmp restart  # 容器重启
$ lnmp kill     # 删除容器
```

## 目录
```
~/dockerapps/
    lnmp/
        data/   # 数据文件夹
        logs/   # 日志文件夹
        files/
            filebeat/
	            Dockerfile
	            filebeat.yml
            memcached/
                Dockerfile
            mongo/
                Dockerfile
                mongod.conf
            mysql/
                Dockerfile
                my.cnf
            nginx/
                conf.d/
                    default.conf
                Dockerfile
                nginx.conf
            php56/
                pkg/
                    memcache.taz # php7 memcache扩展
                Dockerfile
                php.ini
                php-dev.ini
                php-fpm.conf
            php72/
                pkg/
                Dockerfile
                php.ini
                php-dev.ini
                php-fpm.conf
            redis/
                Dockerfile	
            docker-compose.yml  # 基础的nginx+php56+php72+mysql+reids+mongo+memcached配置
            docker-lnmp72.yml    # 基础的nginx+php72+mysql配置
            docker-lnmp72-filebeat.yml    # 基础的nginx+php72+mysql配置，带nginx日志的收集的分析的脚本，要结合https://github.com/canren/elk使用
        .env        # 配置文件
        lnmp        # 启动脚本
        lnmp72        # 启动脚本
        Readme.md   # 帮助文档
    app/
        index.php
        mongo.php    # mongo测试文件
        mysql.php    # mysql测试文件
        phpinfo.php  # phpinfo
        redis.php    # redis测试文件
        memcache.php # memcache测试文件
    xxx/
    xxx2/
```
地址：[https://github.com/canren/elk](https://github.com/canren/elk)
------------------------

### docker-compose相关
#### docker-compose.yml
```dsconfig
version: '3.7'
services:
  php-fpm56: # 只用php-fpm72可以删除php-fpm56
    build:
      context: php56/
      args:
        LNMP_PHP56_VERSION: $LNMP_PHP56_VERSION
    container_name: "canren-lnmp-php56"
    ports:
      - "9001:9000"
    networks:
      - "lnmp"
    volumes:
      - ../../../dockerapps:/data/www:rw
      - ./php56/php-dev.ini:/usr/local/etc/php/php.ini:ro
      - ./php56/php-fpm.conf:/usr/local/etc/php-fpm.conf:ro
      - ../logs/php-fpm56:/var/log/php-fpm:rw
    restart: always
    command: php-fpm

  php-fpm72:
    build:
      context: php72/
      args:
        LNMP_PHP72_VERSION: $LNMP_PHP72_VERSION
    container_name: "canren-lnmp-php72"
    ports:
      - "9000:9000"
    networks:
      - "lnmp"
    volumes:
      - ../../../dockerapps:/data/www:rw
      - ./php72/php-dev.ini:/usr/local/etc/php/php.ini:ro
      - ./php72/php-fpm.conf:/usr/local/etc/php-fpm.conf:ro
      - ../logs/php-fpm72:/var/log/php-fpm:rw
    restart: always
    command: php-fpm

  nginx:
    build:
      context: nginx/
      args:
        LNMP_NGINX_VERSION: $LNMP_NGINX_VERSION
    container_name: "canren-lnmp-nginx"
    ports:
      - "80:80"
    depends_on:
      - php-fpm72
    networks:
      - "lnmp"
    volumes:
      - ../../../dockerapps:/data/www:rw
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ../logs/nginx:/var/log/nginx:rw
    restart: always
    command: nginx -g 'daemon off;'

  mysql-db:
    build:
      context: mysql/
      args:
        LNMP_MYSQL_VERSION: $LNMP_MYSQL_VERSION
    container_name: "canren-lnmp-mysql"
    ports:
      - "3306:3306"
    networks:
      - "lnmp"
    volumes:
      - ../data/mysql:/var/lib/mysql:rw
      - ../logs/mysql:/var/lib/mysql-logs:rw
      - ./mysql/my.cnf:/etc/mysql/conf.d/my.cnf:ro
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: db_test
      MYSQL_USER: test
      MYSQL_PASSWORD: test
    restart: always
    command: "--character-set-server=utf8"

  redis-db:
    build:
      context: redis/
      args:
        LNMP_REDIS_VERSION: $LNMP_REDIS_VERSION
    container_name: "canren-lnmp-redis"
    ports:
      - "6379:6379"
    networks:
      - "lnmp"
    volumes:
      - ../data/redis:/data
    restart: always

  mongo-db:
    build:
      context: mongo/
      args:
        LNMP_MONGO_VERSION: $LNMP_MONGO_VERSION
    container_name: "canren-lnmp-mongo"
    ports:
      - "27017:27017"
    networks:
      - "lnmp"
    volumes:
      - ../data/mongo:/data:rw
      - ../logs/mongo:/var/log/mongodb:rw
      - ./mongo/mongod.conf:/etc/mongod.conf:ro
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 123456
    restart: always

  memcached:
    build:
      context: memcached/
      args:
        LNMP_MEMCACHED_VERSION: $LNMP_MEMCACHED_VERSION
    container_name: "canren-lnmp-memcached"
    ports:
      - "11211:11211"
    networks:
      - "lnmp"
    command: "-m 128"
    restart: always

networks:
  lnmp:
    driver: bridge
```

#### docker-lnmp72.yml
```dsconfig
version: '3.7'
services:

  php-fpm72:
    build:
      context: php72/
      args:
        LNMP_PHP72_VERSION: $LNMP_PHP72_VERSION
    container_name: "canren-lnmp-php72"
    ports:
      - "9000:9000"
    networks:
      - "lnmp"
    volumes:
      - ../../../dockerapps:/data/www:rw
      - ./php72/php-dev.ini:/usr/local/etc/php/php.ini:ro
      - ./php72/php-fpm.conf:/usr/local/etc/php-fpm.conf:ro
      - ../logs/php-fpm72:/var/log/php-fpm:rw
    restart: always
    command: php-fpm

  nginx:
    build:
      context: nginx/
      args:
        LNMP_NGINX_VERSION: $LNMP_NGINX_VERSION
    container_name: "canren-lnmp-nginx"
    ports:
      - "80:80"
    depends_on:
      - php-fpm72
    networks:
      - "lnmp"
    volumes:
      - ../../../dockerapps:/data/www:rw
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ../logs/nginx:/var/log/nginx:rw
    restart: always
    command: nginx -g 'daemon off;'

  mysql-db:
    build:
      context: mysql/
      args:
        LNMP_MYSQL_VERSION: $LNMP_MYSQL_VERSION
    container_name: "canren-lnmp-mysql"
    ports:
      - "3306:3306"
    networks:
      - "lnmp"
    volumes:
      - ../data/mysql:/var/lib/mysql:rw
      - ../logs/mysql:/var/lib/mysql-logs:rw
      - ./mysql/my.cnf:/etc/mysql/conf.d/my.cnf:ro
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: db_test
      MYSQL_USER: test
      MYSQL_PASSWORD: test
    restart: always
    command: "--character-set-server=utf8"

networks:
  lnmp:
    driver: bridge
```

#### docker-lnmp72-filebeat.yml
```dsconfig
version: '3.7'
services:

  php-fpm72:
    build:
      context: php72/
      args:
        LNMP_PHP72_VERSION: $LNMP_PHP72_VERSION
    container_name: "canren-lnmp-php72"
    ports:
      - "9000:9000"
    networks:
      - "lnmp"
    volumes:
      - ../../../dockerapps:/data/www:rw
      - ./php72/php-dev.ini:/usr/local/etc/php/php.ini:ro
      - ./php72/php-fpm.conf:/usr/local/etc/php-fpm.conf:ro
      - ../logs/php-fpm72:/var/log/php-fpm:rw
    restart: always
    command: php-fpm

  nginx:
    build:
      context: nginx/
      args:
        LNMP_NGINX_VERSION: $LNMP_NGINX_VERSION
    container_name: "canren-lnmp-nginx"
    ports:
      - "80:80"
    depends_on:
      - php-fpm72
    networks:
      - "lnmp"
    volumes:
      - ../../../dockerapps:/data/www:rw
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ../logs/nginx:/var/log/nginx:rw
    restart: always
    command: nginx -g 'daemon off;'

  mysql-db:
    build:
      context: mysql/
      args:
        LNMP_MYSQL_VERSION: $LNMP_MYSQL_VERSION
    container_name: "canren-lnmp-mysql"
    ports:
      - "3306:3306"
    networks:
      - "lnmp"
    volumes:
      - ../data/mysql:/var/lib/mysql:rw
      - ../logs/mysql:/var/lib/mysql-logs:rw
      - ./mysql/my.cnf:/etc/mysql/conf.d/my.cnf:ro
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: db_test
      MYSQL_USER: test
      MYSQL_PASSWORD: test
    restart: always
    command: "--character-set-server=utf8"

  filebeat:
    build:
      context: filebeat/
      args:
        ELK_VERSION: $ELK_VERSION
    container_name: "canren-lnmp-filebeat"
    depends_on:
      - nginx
    networks:
      - "lnmp"
      - "elk"
    volumes:
      - ../logs/nginx:/var/log/nginx:ro
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
    restart: always

networks:
  lnmp:
    driver: bridge
  elk:
    driver: bridge
```
--------------------

### Dockerfile相关
#### php5.6
>  php5.6 Decokerfile相关
```dsconfig
ARG LNMP_PHP56_VERSION

FROM php:$LNMP_PHP56_VERSION
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/shanghai
RUN ln -snf /usr/share/zone/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 更新安装依赖包和php核心扩展
RUN apt-get update && apt-get install -y git libfreetype6-dev libjpeg62-turbo-dev libpng-dev \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd zip pdo_mysql opcache mysqli \
    && rm -r /var/lib/apt/lists/*

RUN pecl install redis && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini
RUN pecl install mongodb && echo "extension=mongodb.so" > /usr/local/etc/php/conf.d/mongodb.ini
RUN pecl install memcache && echo "extension=memcache.so" > /usr/local/etc/php/conf.d/memcache.ini

# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH

WORKDIR /data/www

RUN usermod -u 1000 www-data
```
#### php7.2
>  php7.2 Decokerfile相关
```dsconfig
ARG LNMP_PHP72_VERSION

FROM php:$LNMP_PHP72_VERSION
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
```
#### nginx
>  nginx Decokerfile相关
```dsconfig
ARG LNMP_NGINX_VERSION

FROM nginx:$LNMP_NGINX_VERSION
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /data/www
```
#### redis
>  reids Decokerfile相关
```dsconfig
ARG LNMP_REDIS_VERSION

FROM redis:$LNMP_REDIS_VERSION
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
#### mysql
>  mysql Decokerfile相关
```dsconfig
ARG LNMP_MYSQL_VERSION

FROM mysql:$LNMP_MYSQL_VERSION
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
#### mongo
>  mongo Decokerfile相关
```dsconfig
ARG LNMP_MONGO_VERSION

FROM mongo:$LNMP_MONGO_VERSION
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
#### memcached
>  memcached Decokerfile相关
```dsconfig
ARG LNMP_MEMCACHED_VERSION

FROM memcached:$LNMP_MEMCACHED_VERSION
MAINTAINER canren "bestsxf@gmail.com"
```
#### filebeat
>  filebeat Decokerfile相关
```dsconfig
ARG ELK_VERSION

FROM docker.elastic.co/beats/filebeat:$ELK_VERSION
MAINTAINER canren "bestsxf@gmail.com"
```
