# 创建基于Alpine Linux的JRE环境

## Alpine的更新源

Alpine的更新源配置文件repositories内容,如果自建源，可以替换成本地的源，国外源实在太慢。

```
$ cat repositories

http://mirrors.aliyun.com/alpine/v3.5/main/
http://mirrors.aliyun.com/alpine/v3.5/community/
http://mirrors.aliyun.com/alpine/v3.5/releases/
```

## Dockerfile 

```
FROM alpine:3.5

MAINTAINER cloudtogo

ENV JAVA_HOME=/usr/lib/jvm/default-jvm/jre

ADD repositories /etc/apk/repositories

RUN apk upgrade --update-cache; \
    apk add openjdk8-jre; \
    rm -rf /tmp/* /var/cache/apk/*

CMD ["java", "-version"]
```


build:

```
docker build -t registry.cn-hangzhou.aliyuncs.com/cloudtogo/alpine-openjdk:8-jre .
```