ARG REDIS_VERSION

FROM redis:5

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# RUN sed -i 's/# requirepass/requirepass 123456/g' /etc/apk/repositories \
