# Docker构建自己的PHP开发环境

[TOC]
## 使用

### linux 环境
```bash
$ mkdir ~/dockerapps # ln -s 自己的工作目录过来
$ cd ~/dockerapps
$ git clone https://github.com/canren/lnmp.git
$ cd lnmp
$ ./lnmp build # 中间断开命令重试即可
```

### linux 环境
```
windows环境脚本还在编写当中
```
------------------------
### 容器相关命令

> container.sh shell
```bash
~/dockerapps/lnmp # shell路径
$ container.sh build # 容器打包
$ container.sh start # 开启容器
$ container.sh stop # 关闭容器
$ container.sh restart # 容器重启
$ container.sh kill # 删除容器
```

## 目录
```
~/dockerapps/
	lnmp/
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
		docker-compose.yml
		container.sh
	app/
		index.php
		mongo.php
		mysql.php
		phpinfo.php
		redis.php
	xxx/
	xxx2/
```

------------------------

### docker-compose相关
#### docker-compose.yml
```dsconfig
version: '3.7'
services:
  php-fpm56: # 只用php-fpm72可以删除php-fpm56配置
    build: ./php56
    container_name: "lnmp-php56"
    ports:
      - "9001:9000"
    volumes:
      - ../../../dockerapps:/data/www:rw
      - ./php56/php-dev.ini:/usr/local/etc/php/php.ini:ro
      - ./php56/php-fpm.conf:/usr/local/etc/php-fpm.conf:ro
      - ../logs/php-fpm56:/var/log/php-fpm:rw
    restart: always
    command: php-fpm

  php-fpm72: # 只用php-fpm56可以删除php-fpm72配置
    build: ./php72
    container_name: "lnmp-php72"
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
    build: ./nginx
    container_name: "lnmp-nginx"
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
    build: ./mysql
    container_name: "lnmp-mysql"
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

  redis-db: # 不用redis可以删除配置
    build: ./redis
    container_name: "lnmp-redis"
    ports:
      - "6379:6379"
    networks:
      - "lnmp"
    volumes:
      - ../data/redis:/data
    restart: always

  mongo-db: # 不用mongo可以删除配置
    build: ./mongo
    container_name: "lnmp-mongo"
    ports:
      - "27017:27017"
    volumes:
      - ../data/mongo:/data:rw
      - ../logs/mongo:/var/log/mongodb:rw
      - ./mongo/mongod.conf:/etc/mongod.conf:ro
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 123456
    command: mongod

networks:
  lnmp:
    driver: bridge
```
--------------------

### Dockerfile相关
#### php5.6
>  php5.6 Decokerfile相关
```dsconfig
FROM php:5.6-fpm
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/shanghai
RUN ln -snf /usr/share/zone/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 更新安装依赖包和php核心扩展
RUN apt-get update && apt-get install -y \
    git \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install zip \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install opcache \
    && docker-php-ext-install mysqli \
    && rm -r /var/lib/apt/lists/*

RUN pecl install redis && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini
RUN pecl install mongodb && echo "extension=mongodb.so" > /usr/local/etc/php/conf.d/mongodb.ini

# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH

WORKDIR /data

RUN usermod -u 1000 www-data
```
#### php7.2
>  php7.2 Decokerfile相关
```dsconfig
FROM php:7.2-fpm
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/shanghai
RUN ln -snf /usr/share/zone/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 更新安装依赖包和php核心扩展
RUN apt-get update && apt-get install -y \
    git \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install zip \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install opcache \
    && docker-php-ext-install mysqli \
    && rm -r /var/lib/apt/lists/*

RUN pecl install redis && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini
RUN pecl install mongodb && echo "extension=mongodb.so" > /usr/local/etc/php/conf.d/mongodb.ini

# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH

WORKDIR /data

RUN usermod -u 1000 www-data
```
#### nginx
>  nginx Decokerfile相关
```dsconfig
FROM nginx:1.15-alpine
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
#### mysql
>  mysql Decokerfile相关
```dsconfig
FROM mysql:5.7
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
#### redis
>  reids Decokerfile相关
```dsconfig
FROM redis:5-alpine
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
#### mongo
>  mongo Decokerfile相关
```dsconfig
FROM mongo:3.6-xenial
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
