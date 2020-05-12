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
$ docker-compose down    # Stop and Remove services
$ docker exec -it ${NAMES} bash    # 进入docker镜像
```
## 目录
```
~/dockerapps/
    lnmp/
        data/   # 数据文件夹
        logs/   # 日志文件夹
        files/
            mysql/
                Dockerfile
                my.cnf
            neo4j/
                conf/
                    neo4j.conf
                Dockerfile
            openresty/ # 基于 NGINX 和 LuaJIT 的 Web 平台
                conf.d/
                    default.conf
                Dockerfile
                nginx.conf
            php72/
                pkg/
                Dockerfile
                php.ini
                php-dev.ini
                php-fpm.conf
            redis/ # 默认密码为123456
                Dockerfile	
            docker-compose.yml  # 基础的openresty+php72+mysql+reids配置
        .env.production        # 配置文件
        Readme.md   # 帮助文档
```