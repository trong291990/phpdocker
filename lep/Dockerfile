FROM alpine:3.18

ARG TZ=Asia/Tokyo
ARG USER_ID=501
ARG GROUP_ID=20
ENV USER_ID=${USER_ID} \
    GROUP_ID=${GROUP_ID} \
    TERM=xterm \
    TZ=Asia/Tokyo \
    ALPINE_FRONTEND=noninteractive

RUN apk update && \
    apk add --no-cache \
    tzdata \
    nginx \
    supervisor \
    curl \
    git \
    zip \
    unzip \
    wget \
    nano \
    libreoffice \
    php82 \
    php82-bcmath \
    php82-bz2 \
    php82-calendar \
    php82-ctype \
    php82-curl \
    php82-dom \
    php82-fpm \
    php82-gd \
    php82-gettext \
    php82-iconv \
    php82-intl \
    php82-mbstring \
    php82-mysqli \
    php82-mysqlnd \
    php82-opcache \
    php82-pdo \
    php82-pdo_mysql \
    php82-pdo_pgsql \
    php82-pdo_sqlite \
    php82-pgsql \
    php82-phar \
    php82-session \
    php82-simplexml \
    php82-sqlite3 \
    php82-tokenizer \
    php82-xml \
    php82-xmlwriter \
    php82-zip \
    php82-zlib

# Set timezone
RUN cp /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo ${TZ} > /etc/timezone

# Config PHP
RUN ln -s /usr/bin/php82 /usr/bin/php
RUN mkdir /run/php
COPY php-fpm/8.2/www.conf /etc/php82/php-fpm.d/www.conf
COPY php-fpm/8.2/php.ini /etc/php82/php.ini

# Configure Nginx
RUN if ! getent group www-data > /dev/null 2>&1; then \
    addgroup -S www-data; \
fi && \
if ! getent passwd www-data > /dev/null 2>&1; then \
    adduser -S -D -H -G www-data www-data; \
fi
RUN mkdir -p /run/nginx && chown www-data:www-data /run/nginx
RUN mkdir -p /var/www/html && echo "<?php phpinfo();" > /var/www/html/info.php
RUN echo "<h1>Nginx Works!</h1><a href='info.php'>PHP Info</a>" > /var/www/html/index.html

COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY nginx/sites /etc/nginx/sites-enabled

# Cronjob
COPY cron/laravel /etc/cron.d/laravel
RUN chmod 0644 /etc/cron.d/laravel && \
    crontab /etc/cron.d/laravel

# Supervisor
RUN mkdir -p /var/log/supervisor
COPY supervisor/supervisord.conf /etc/supervisord.conf

# Composer
RUN curl -sS https://getcomposer.org/installer | php82 -- --install-dir=/usr/local/bin --filename=composer

# Adminer
RUN mkdir /usr/share/adminer
COPY vendor/adminer-4.8.1.php /usr/share/adminer/index.php

RUN apk add php82-fileinfo php82-xmlreader

# Clean up
RUN rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

# Copy startup script
COPY ./docker-run.sh /docker-run.sh
RUN chmod +x /docker-run.sh

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]

WORKDIR /var/www/Workspace

EXPOSE 80 443 6379 3306