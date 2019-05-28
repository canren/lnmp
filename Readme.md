# Docker构建自己的PHP开发环境

[TOC]
## 使用
###
> php项目路径设置 .env文件WORKSPACES参数 默认=../../../dockerapps:/data/www:rw 相对路径、绝对路径均可
### linux 环境
> 下面是我写的几个dockerfile启动脚本，如果自己有需要可以自己搭配环境
#### 基础的nginx+php72+mysql+reids配置
```bash
$ mkdir ~/dockerapps
$ cd ~/dockerapps
$ git clone https://github.com/canren/lnmp.git
$ cd lnmp
$ cp .env.production .env # 必须修改WORKSPACES为自己的工作目录
$ chmod -R 0755 ./lnmp
$ ./lnmp build # 中间断开命令重试即可
```
#### 基础的nginx+php72+mysql配置
```bash
$ mkdir ~/dockerapps
$ cd ~/dockerapps
$ git clone https://github.com/canren/lnmp.git
$ cd lnmp
$ cp .env.production .env # 必须修改WORKSPACES为自己的工作目录
$ chmod -R 0755 ./lnmp72
$ ./lnmp72 build # 中间断开命令重试即可
```
#### 基础的nginx+php72+mysql配置，带nginx日志的收集的分析的脚本
>这个需要首先配置elk docker脚本,相关[https://github.com/canren/elk](https://github.com/canren/elk)，`elk脚本启动完成之后，再启动该脚本`
```bash
$ mkdir ~/dockerapps
$ cd ~/dockerapps
$ git clone https://github.com/canren/lnmp.git
$ cd lnmp
$ cp .env.production .env # 必须修改WORKSPACES为自己的工作目录
$ chmod -R 0755 ./lnmp72_filebeat
$ ./lnmp72_filebeat build # 中间断开命令重试即可
```
#### 基础的nginx+php56+php72+mysql+reids+mongo+memcached配置
```bash
$ mkdir ~/dockerapps
$ cd ~/dockerapps
$ git clone https://github.com/canren/lnmp.git
$ cd lnmp
$ cp .env.production .env # 必须修改WORKSPACES为自己的工作目录
$ chmod -R 0755 ./lnmp_all
$ ./lnmp_all build # 中间断开命令重试即可
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
$ lnmp test     # 容器测试，不会生成images镜像
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
            redis/ # 默认密码为123456
                Dockerfile	
            docker-all.yml  # 基础的nginx+php56+php72+mysql+reids+mongo+memcached配置
            docker-compose.yml  # 基础的nginx+php72+mysql+reids配置
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