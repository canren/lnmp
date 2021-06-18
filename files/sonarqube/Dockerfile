ARG REDIS_VERSION

FROM sonarqube:latest
MAINTAINER canren "bestsxf@gmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV JAVA_VERSION=jdk-11.0.8+10 LANG=en_US.UTF-8 LANGUAGE=en_US:en

WORKDIR /opt/sonarqube


ENTRYPOINT ["./bin/run.sh"]