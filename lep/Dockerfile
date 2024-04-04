FROM ubuntu:22.04


ARG TZ=Asia/Tokyo
ARG USER_ID=501
ARG GROUP_ID=20
ENV USER_ID ${USER_ID}
ENV GROUP_ID ${GROUP_ID}

ENV TERM=xterm \
    TZ=Asia/Tokyo \
    DEBIAN_FRONTEND=noninteractive


RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update --fix-missing  && \
    apt-get install -y software-properties-common

RUN add-apt-repository ppa:ondrej/php
RUN apt-get update

# Install apps mysql-server
RUN apt-get -y install python3-pip  supervisor nginx curl git cron  zip unzip curl wget nano  \
    php8.1-cli php8.1-common php8.1-mysql php8.1-zip php8.1-gd php8.1-mbstring php8.1-curl php8.1-xml php8.1-fpm php8.1-bcmath \
    php7.4-cli php7.4-common php7.4-mysql php7.4-zip php7.4-gd php7.4-mbstring php7.4-curl php7.4-xml php7.4-fpm php7.4-bcmath php7.4-imagic php7.4-intl

# Python package

RUN apt-get -y install php8.1-pgsql php7.4-pgsql

# PHP-FPM
RUN mkdir /run/php
COPY php-fpm/8.1/www.conf /etc/php/8.1/fpm/pool.d/www.conf
COPY php-fpm/8.1/php.ini /etc/php/8.1/fpm/php.ini

COPY php-fpm/7.4/www.conf /etc/php/7.4/fpm/pool.d/www.conf
COPY php-fpm/7.4/php.ini /etc/php/7.4/fpm/php.ini

#  Nginx
RUN mkdir -p /var/www/html && echo "<?php phpinfo();" > /var/www/html/index.php
COPY nginx/nginx.conf /etc/nginx/nginx.conf
RUN rm /etc/nginx/sites-enabled/default
COPY nginx/sites /etc/nginx/sites-enabled

# Cronjob
COPY cron/laravel /etc/cron.d/laravel
RUN chmod 0644 /etc/cron.d/laravel
RUN crontab /etc/cron.d/laravel

# Supervisor
COPY supervisor/supervisord.conf /etc/supervisor/supervisord.conf


# Composer
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php
RUN php composer-setup.php --install-dir=/usr/local/bin --filename=composer

# Adminer
RUN mkdir /usr/share/adminer
COPY vendor/adminer-4.8.1.php /usr/share/adminer/index.php

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog


COPY ./docker-run.sh /docker-run.sh

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]


EXPOSE 80 443 6379 3306
