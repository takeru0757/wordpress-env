FROM php:7.2.14-fpm-alpine3.8

# ------------------------------------------------------------------------------
# Locale / Timezone / certificates
# ------------------------------------------------------------------------------

# Setting `$TZ` will reset timezone because `$TZ` depends tzdata.
# Do not set `$TZ` if delete tzdata package.

ENV LC_ALL C.UTF-8

RUN set -ex \
  && apk add --no-cache \
    tzdata \
    ca-certificates \
  && cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
  && echo "Asia/Tokyo" > /etc/timezone \
  && apk del tzdata \
  && update-ca-certificates

# ------------------------------------------------------------------------------
# APP_ROOT, APP_USER (+ sudo)
# ------------------------------------------------------------------------------

ENV APP_ROOT=/var/www/html \
    APP_USER=www-data

RUN set -ex \
  && mkdir -p "${APP_ROOT}" \
  && chown -R "${APP_USER}:${APP_USER}" "${APP_ROOT}" \
  && apk add --no-cache sudo \
  && echo "${APP_USER} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# ------------------------------------------------------------------------------
# Utilities
# - su-exec (https://github.com/ncopa/su-exec)
# - dockerize (https://github.com/jwilder/dockerize)
# ------------------------------------------------------------------------------

ENV DOCKERIZE_VERSION v0.6.1

RUN set -ex \
  && apk add --no-cache \
    openssl \
    su-exec \
  && wget "https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz" \
  && tar -C /usr/local/bin -xzvf "dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz" \
  && rm "dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz"

# ------------------------------------------------------------------------------
# nginx (+ php-fpm)
# ------------------------------------------------------------------------------

ENV APK_NGINX_VERSION 1.14.2-r0

RUN set -ex \
  && apk add --no-cache nginx="${APK_NGINX_VERSION}"

COPY php-fpm.d/*.conf /usr/local/etc/php-fpm.d/
COPY nginx/*.conf /etc/nginx/
COPY nginx/conf.d/*.conf /etc/nginx/conf.d/

# ------------------------------------------------------------------------------
# supervisor
# ------------------------------------------------------------------------------

ENV APK_SUPERVISOR_VERSION 3.3.4-r1

RUN set -ex \
  && apk add --no-cache supervisor="${APK_SUPERVISOR_VERSION}"

COPY supervisor/supervisord.conf /etc/supervisor/
COPY supervisor.d/*.ini /etc/supervisor.d/

# ------------------------------------------------------------------------------
# php
# ------------------------------------------------------------------------------

ENV XDEBUG_VERSION=2.6.1

RUN set -ex \
  && apk add --no-cache --virtual .php-xdebug-build-deps \
    autoconf \
    g++ \
    make \
  && pecl install "xdebug-${XDEBUG_VERSION}" \
  && docker-php-ext-enable xdebug \
  && apk del .php-xdebug-build-deps

RUN set -ex \
  && apk add --no-cache \
    icu-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    zlib-dev \
  && docker-php-ext-configure gd \
    --with-freetype-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
  && docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) \
    intl \
    opcache \
    mysqli \
    gd \
    zip

COPY php/conf.d/*.ini /usr/local/etc/php/conf.d/

# ------------------------------------------------------------------------------
# ENTRYPOINT / CMD
# ------------------------------------------------------------------------------

COPY docker-entrypoint /usr/local/bin/

WORKDIR "${APP_ROOT}"
USER "${APP_USER}"

EXPOSE 80

ENTRYPOINT ["docker-entrypoint"]
CMD ["sudo", "-E", "supervisord", "-c", "/etc/supervisor/supervisord.conf"]
