# Docker构建自己的PHP开发环境

[TOC]
## 使用
### 项目路径配置
> php项目路径设置 .env文件WORKSPACES参数 默认=../../../dockerapps:/data/www:rw 相对路径、绝对路径均可
### docker-compose 命令
> docker-compose 命令，-f 后面参数跟.yml文件名,例如`docker-compose -f docker-compose.yml up`，默认文件`docker-compose.yml`。详细命令请输入`docker-compose --help`查看
```bash
$ docker-compose up       # Create and start containers
$ docker-compose start    # Start services
$ docker-compose stop     # Stop services
$ docker-compose restart  # Restart services
$ docker-compose kill     # Kill containers
$ docker-compose rm       # Remove stopped containers
$ docker-compose pause    # Pause services
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
            openresty/ # 基于 NGINX 和 LuaJIT 的 Web 平台
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
            redis/ # 默认密码为123456
                Dockerfile	
            docker-all.yml  # 基础的openresty+nginx+php56+php72+mysql+reids+mongo+memcached配置
            docker-compose.yml  # 基础的openresty+php72+mysql+reids配置
            docker-lnmp72.yml    # 基础的openresty+php72+mysql配置
            docker-lnmp72-filebeat.yml    # 基础的openresty+php72+mysql配置，带nginx日志的收集的分析的脚本，要结合https://github.com/canren/elk使用
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