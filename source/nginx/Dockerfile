FROM nginx:1.15.5
RUN sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list && \
    rm -Rf /var/lib/apt/lists/* && \
    apt-get update && apt-get install -y vim git
#时区
ENV TZ Asia/Shanghai
RUN date -R
#容器启动时执行指令
CMD ["nginx", "-g", "daemon off;"]