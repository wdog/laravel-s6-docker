FROM php:8.3.0-fpm-alpine3.19

WORKDIR /var/www/html

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

RUN apk add nginx s6-overlay

# ------------------------ Common PHP Dependencies ------------------------
RUN apk update && apk add \
    # see https://github.com/docker-library/php/issues/880
    oniguruma-dev \
    # needed for gd
    libpng-dev libjpeg-turbo-dev icu-dev \
    # number formatter locales
    icu-data-full \
    # needed for xdebug
    $PHPIZE_DEPS \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    # Installing common Laravel dependencies
    && docker-php-ext-configure intl \
    && docker-php-ext-install mbstring pdo_mysql gd bcmath intl \
    opcache \
    && mkdir -p /home/www-data/.composer/cache \
    && chown -R www-data:www-data /home/www-data/ /var/www/html

RUN docker-php-ext-configure pcntl --enable-pcntl \
  && docker-php-ext-install pcntl;

RUN apk add icu-dev libzip-dev && docker-php-ext-install zip bz2 intl
# ------------------------ xdebug ------------------------
ARG XDEBUG_ENABLE=false
ARG REMOTE_XDEBUG_HOST=0.0.0.0
ARG REMOTE_XDEBUG_PORT=9999
RUN if [ "$XDEBUG_ENABLE" = "true" ]; then \
    pecl install xdebug-3.0.0; \
    docker-php-ext-enable xdebug; \
    echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_autostart=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_host=${REMOTE_XDEBUG_HOST}" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_port=${REMOTE_XDEBUG_PORT}" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_handler=dbgp" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_connect_back=On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.idekey=x_debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    mv /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini.disabled; \
    fi

# ------------------------ start fpm/nginx ------------------------


RUN apk add openssl && openssl req -x509 -nodes -days 365 \
    -subj  "/C=IT/ST=QC/O=dark Empire/CN=localhost" \
    -newkey rsa:2048 -keyout /etc/ssl/private/nginx.key \
    -out /etc/ssl/certs/nginx.crt;

COPY .docker/services.d /etc/services.d
COPY .docker/nginx.conf /etc/nginx/nginx.conf
COPY .docker/default.conf /etc/nginx/conf.d/default.conf

# change uid to current user uid ( id -u ) (id -g)
ARG UID=1000
ARG GID=1000
RUN apk add shadow && usermod -u $UID www-data && groupmod -g $GID www-data

# Adding the opcache configuration into the wrong directory intentionally.
# This is enabled at runtime
ADD .docker/opcache.ini          /usr/local/etc/php/opcache.ini
ADD .docker/opcache_disabled.ini /usr/local/etc/php/opcache_disabled.ini
ADD .docker/php_extra.ini        /usr/local/etc/php/conf.d/php_extra.ini

ARG APP_ENV=local
RUN if [ "$APP_ENV" == "production" ]; then \
    mv /usr/local/etc/php/opcache.ini /usr/local/etc/php/conf.d/opcache.ini; \
    else \
    mv /usr/local/etc/php/opcache_disabled.ini /usr/local/etc/php/conf.d/opcache.ini; \
    fi


ADD .docker/healthcheck.ini /usr/local/etc/php/healthcheck.ini

## DEV
RUN if [ "$VITE_ENABLED" = "true" ]; then \
    apk add nodejs npm; \
    fi

## TEST
RUN apk add nodejs npm;

RUN rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*

EXPOSE 80 443 5173

ENTRYPOINT ["/init"]
CMD []
