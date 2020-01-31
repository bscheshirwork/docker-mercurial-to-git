FROM python:2.7-alpine3.11
# docker build --pull --no-cache -t bscheshir/hg-to-git:python2.7-alpine3.11 -- ./build && docker push bscheshir/hg-to-git:python2.7-alpine3.11
MAINTAINER BSCheshir <bscheshir.work@gmail.com>

RUN apk --update add \
        git \
        mercurial \
        git-fast-import \
        bash \
        bash-completion

# Prepare host-volume working directory
RUN mkdir -p /data
WORKDIR /data

RUN git clone https://github.com/frej/fast-export.git /data/fast-export
