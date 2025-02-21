FROM php:8.3.16-fpm-alpine3.21 AS base

WORKDIR /var/www/html

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

RUN apk add nginx s6-overlay

# Installa le dipendenze comuni
RUN apk add $PHPIZE_DEPS \
    && docker-php-ext-install pdo_mysql


# number formatter locales icu-data-full
# oniguma see https://github.com/docker-library/php/issues/880

RUN apk add --no-cache --update \
    shadow \
    openssl \
    oniguruma-dev \
    gpg curl \
    sqlite-dev \
    libpng-dev libjpeg-turbo-dev icu-dev \
    icu-data-full libzip-dev && \
    docker-php-ext-install pcntl mbstring gd bcmath zip bz2 intl && \
    docker-php-ext-configure intl && \
    docker-php-ext-configure pcntl --enable-pcntl && \
    mkdir -p /home/www-data/.composer/cache && \
    chown -R www-data:www-data /home/www-data/ /var/www/html ;


RUN openssl req -x509 -nodes -days 365 \
    -subj  "/C=IT/ST=QC/O=dark Empire/CN=localhost" \
    -newkey rsa:2048 -keyout /etc/ssl/private/nginx.key \
    -out /etc/ssl/certs/nginx.crt;

COPY .docker/services.d /etc/services.d
COPY .docker/nginx.conf /etc/nginx/nginx.conf
COPY .docker/default.conf /etc/nginx/conf.d/default.conf


# -----------
# SQL SERVER
# -----------
ENV ACCEPT_EULA=Y
# Install prerequisites for the sqlsrv and pdo_sqlsrv PHP extensions.
RUN cd /tmp && \
    curl -O https://download.microsoft.com/download/7/6/d/76de322a-d860-4894-9945-f0cc5d6a45f8/msodbcsql18_18.4.1.1-1_amd64.apk && \
    curl -O https://download.microsoft.com/download/7/6/d/76de322a-d860-4894-9945-f0cc5d6a45f8/mssql-tools18_18.4.1.1-1_amd64.apk && \
    apk add --allow-untrusted msodbcsq*.apk && \
    apk add --allow-untrusted mssql-tools*.apk && \
    apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS unixodbc-dev && \
    pecl channel-update pecl.php.net && \
    pecl install sqlsrv pdo_sqlsrv && \
    docker-php-ext-enable sqlsrv pdo_sqlsrv && \
    apk del .phpize-deps && \
    rm -rf /tmp/*.apk && \
    rm -rf /var/cache/apk/* && \
    cd -



# change uid to current user uid ( id -u ) (id -g)
ARG UID=1000
ARG GID=1000
RUN usermod -u $UID www-data && \
    groupmod -g $GID www-data ;

# TO BUILD ASSETS ALSO IN PRODUCTION
RUN apk add nodejs npm;

# RUN rm -rf /var/cache/apk/* && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* ;

ADD .docker/php_extra.ini        /usr/local/etc/php/conf.d/php_extra.ini
ADD .docker/healthcheck.ini /usr/local/etc/php/healthcheck.ini

RUN rm -rf /var/cache/apk/*
EXPOSE 80 443 5173

# --------------------------
# DEVELOP ENV
# --------------------------

FROM base AS local
# ------- xdebug -------
# ARG XDEBUG_ENABLED=false

RUN pecl channel-update pecl.php.net
RUN apk add --update linux-headers
# ! TODO FIX
# RUN pecl install -o -f xdebug-3.4.0 && docker-php-ext-enable xdebug


# RUN if [ "$XDEBUG_ENABLED" = "true" ]; then \
#     pecl channel-update pecl.php.net && \
#     apk add --update linux-headers && \
#     pecl install xdebug-3.4.0 ; \
# fi


# COPY .docker/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

ENTRYPOINT ["/init"]
CMD []


# --------------------------
# PRODUCTION
# --------------------------

FROM base AS production

RUN docker-php-ext-install opcache

ADD .docker/opcache.ini /usr/local/etc/php/opcache.ini
# ------- start fpm/nginx ------------------------

RUN apk del .build-deps && rm -rf /var/cache/apk/*
ENTRYPOINT ["/init"]
CMD []
