FROM python:2.7-alpine3.11
# docker build --pull --no-cache -t bscheshir/hg-to-git:python2.7-alpine3.11 -- ./build && docker push bscheshir/hg-to-git:python2.7-alpine3.11
# docker build --pull -t bscheshir/hg-to-git:python2.7-alpine3.11 -- ./build
MAINTAINER BSCheshir <bscheshir.work@gmail.com>

RUN apk --update add \
        git \
        git-fast-import \
        bash \
        bash-completion
RUN apk add --no-cache --virtual .build-deps \
    gcc musl-dev \
    && pip install mercurial \
    && find /usr/local \
        \( -type d -a -name test -o -name tests \) \
        -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
        -exec rm -rf '{}' + \
    && runDeps="$( \
        scanelf --needed --nobanner --recursive /usr/local \
                | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
                | sort -u \
                | xargs -r apk info --installed \
                | sort -u \
    )" \
    && apk add --virtual .rundeps $runDeps \
    && apk del .build-deps

# Prepare host-volume working directory
RUN mkdir -p /data
WORKDIR /data

RUN git clone https://github.com/frej/fast-export.git /data/fast-export
