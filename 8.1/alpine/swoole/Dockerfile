# hyperf/hyperf:8.1
#
# @link     https://www.hyperf.io
# @document https://hyperf.wiki
# @contact  group@hyperf.io
# @license  https://github.com/hyperf/hyperf/blob/master/LICENSE

ARG ALPINE_VERSION

FROM hyperf/hyperf:8.1-alpine-v${ALPINE_VERSION}-base

LABEL maintainer="Hyperf Developers <group@hyperf.io>" version="1.0" license="MIT"

ARG SW_VERSION
ARG COMPOSER_VERSION

##
# ---------- env settings ----------
##
ENV SW_VERSION=${SW_VERSION:-"v4.6.7"} \
    COMPOSER_VERSION=${COMPOSER_VERSION:-"2.0.2"} \
    #  install and remove building packages
    PHPIZE_DEPS="autoconf dpkg-dev dpkg file g++ gcc libc-dev make php8-dev php8-pear pkgconf re2c pcre-dev pcre2-dev zlib-dev libtool automake"

# update
RUN set -ex \
    && apk update \
    # for swoole extension libaio linux-headers
    && apk add --no-cache libstdc++ openssl git bash \
    && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS libaio-dev openssl-dev curl-dev \
    # download
    && cd /tmp \
    && curl -SL "https://github.com/swoole/swoole-src/archive/${SW_VERSION}.tar.gz" -o swoole.tar.gz \
    && ls -alh \
    # php extension:swoole
    && cd /tmp \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && ln -s /usr/bin/phpize8 /usr/local/bin/phpize \
    && ln -s /usr/bin/php-config8 /usr/local/bin/php-config \
    && ( \
        cd swoole \
        && phpize \
        && ./configure --enable-openssl --enable-http2 --enable-swoole-curl --enable-swoole-json \
        && make -s -j$(nproc) && make install \
    ) \
    && echo "memory_limit=1G" > /etc/php8/conf.d/00_default.ini \
    && echo "opcache.enable_cli = 'On'" >> /etc/php8/conf.d/00_opcache.ini \
    && echo "extension=swoole.so" > /etc/php8/conf.d/50_swoole.ini \
    && echo "swoole.use_shortname = 'Off'" >> /etc/php8/conf.d/50_swoole.ini \
    # install composer
    && wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod u+x /usr/local/bin/composer \
    # php info
    && php -v \
    && php -m \
    && php --ri swoole \
    && php --ri Zend\ OPcache \
    && composer \
    # ---------- clear works ----------
    && apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/local/bin/php* \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"
