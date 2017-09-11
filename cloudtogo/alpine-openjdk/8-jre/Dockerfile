FROM alpine:3.5

MAINTAINER cloudtogo

LABEL "alpine.version"=3.5 \
      "jre.version"=openjdk8-jre

ENV JAVA_HOME=/usr/lib/jvm/default-jvm/jre

ADD repositories /etc/apk/repositories

RUN apk upgrade --update-cache; \
    apk add openjdk8-jre; \
    rm -rf /tmp/* /var/cache/apk/*

CMD ["java", "-version"]