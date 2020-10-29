ARG LNMP_OPENRESTY_VERSION

FROM openresty/openresty:$LNMP_OPENRESTY_VERSION
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# COPY ./apt/sources.list /etc/apt/sources.list

# RUN apt-get update
# RUN apt-get install -y snapd
# RUN systemctl enable snapd

# CMD [ "/sbin/init" ]
WORKDIR /data/www