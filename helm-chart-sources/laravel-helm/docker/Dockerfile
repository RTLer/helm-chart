FROM php:8.1-fpm-alpine3.16 as base

RUN set -ex
RUN apk --no-cache add tzdata curl ca-certificates linux-headers

RUN cp /usr/share/zoneinfo/Asia/Tehran /etc/localtime

ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions
RUN install-php-extensions zip soap bcmath pdo_pgsql pcntl intl sockets imap opcache @composer

RUN printf 'opcache.enable_cli=1\n' >> /usr/local/etc/php/conf.d/opcache.ini
RUN printf 'opcache.revalidate_freq=0\n' >> /usr/local/etc/php/conf.d/opcache.ini
RUN printf 'opcache.validate_timestamps=0\n' >> /usr/local/etc/php/conf.d/opcache.ini
RUN printf 'opcache.max_accelerated_files=10000\n' >> /usr/local/etc/php/conf.d/opcache.ini
RUN printf 'opcache.interned_strings_buffer=16\n' >> /usr/local/etc/php/conf.d/opcache.ini
RUN printf 'opcache.fast_shutdown=1\n' >> /usr/local/etc/php/conf.d/opcache.ini

RUN mkdir -p   /.composer /.config && \
    chgrp -R 0 /.composer /.config /var/log /var/run /var/tmp && \
    chmod -R g=u,a+rx /.composer /.config /var/log /var/run /var/tmp && \
    rm -rf /var/cache/apk/*

COPY --from=spiralscout/roadrunner:latest /usr/bin/rr /usr/local/bin/rr

WORKDIR /var/www/html


FROM base as application_fpm

COPY composer.json composer.lock ./
RUN composer install --no-scripts --no-autoloader --prefer-dist --no-dev

COPY ./ /tmp/app
RUN chgrp -R 0 /tmp/app && \
    chmod -R g=u /tmp/app && \
    cp -a /tmp/app/. /var/www/html && \
    rm -rf /tmp/app && \
    composer dump-autoload --classmap-authoritative
RUN chmod -R 777 storage/

ARG BUILD_VERSION=unknown
ENV APP_VERSION=$BUILD_VERSION
VOLUME ["/var/www/html"]
EXPOSE 9000


FROM application_fpm as application
EXPOSE 80
CMD ["php", "artisan", "octane:start", "--host=0.0.0.0", "--port=80", "--rpc-port=1919", "--no-ansi"]


FROM application as queue_worker

CMD ["php", "artisan", "horizon"]


FROM application as schedule

RUN printf 'memory_limit = 5G' > /usr/local/etc/php/conf.d/memory.ini

RUN echo "* * * * * php -d memory_limit=10G /var/www/html/artisan schedule:run >> /dev/null 2>&1" | crontab -

CMD ["/usr/sbin/crond", "-f", "-L", "15"]


FROM nginx:alpine as nginx

COPY --from=application /var/www/html/public /var/www/html/public
COPY ./docker/nginx/default.conf /etc/nginx/conf.d/default.conf


FROM application as development
ENV XDEBUG_MODE=coverage
RUN install-php-extensions xdebug
RUN composer install
