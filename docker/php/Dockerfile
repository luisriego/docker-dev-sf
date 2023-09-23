FROM php:8.2-fpm-buster

ARG UID

# Create user with same permissions as host and some useful stuff
RUN adduser -u ${UID} --disabled-password --gecos "" appuser
RUN mkdir /home/appuser/.ssh
RUN chown -R appuser:appuser /home/appuser/
RUN echo "StrictHostKeyChecking no" >> /home/appuser/.ssh/config
RUN echo "export COLUMNS=300" >> /home/appuser/.bashrc
RUN echo "alias sf=/appdata/www/bin/console" >> /home/appuser/.bashrc

# Install packages and PHP extensions
RUN apt update && apt install -y \
     gnupg \
     g++ \
     procps \
     openssl \
     git \
     acl \
     openssl \
     openssh-client \
     wget \
     zip \
     vim \
     unzip \
     zlib1g-dev \
     libzip-dev \
     libfreetype6-dev \
     libpng-dev \
     libjpeg-dev \
     libicu-dev  \
     libonig-dev \
     libxslt1-dev \
     acl \
     # for MySQL
     && docker-php-ext-install pdo_mysql \
     # for Postgres
     && apt install -y libpq-dev \
     && docker-php-ext-install pdo_pgsql \
     # for RabbitMQ and XDEBUG
     && docker-php-ext-install bcmath sockets \
     && apt install -y librabbitmq-dev vim libssh-dev \
     && pecl install xdebug amqp-1.11.0RC1 \
     # XDEBUG and APCu
     && pecl install xdebug apcu \
     && docker-php-ext-enable --ini-name 05-opcache.ini opcache xdebug amqp apcu

 RUN docker-php-ext-configure gd --with-jpeg --with-freetype

 RUN docker-php-ext-install \
     pdo pdo_mysql zip xsl gd intl opcache exif mbstring

# Install and update composer
RUN curl https://getcomposer.org/composer.phar -o /usr/bin/composer && chmod +x /usr/bin/composer
RUN composer self-update

## Install Symfony binary
USER appuser
RUN wget https://get.symfony.com/cli/installer -O - | bash
USER root
# for symfony <= 4
# RUN mv /home/appuser/.symfony/bin/symfony /usr/local/bin/symfony
# for symfony >= 5
RUN mv /home/appuser/.symfony5/bin/symfony /usr/local/bin/symfony

# Install Redis.
RUN \
  cd /tmp && \
  wget http://download.redis.io/redis-stable.tar.gz && \
  tar xvzf redis-stable.tar.gz && \
  cd redis-stable && \
  make && \
  make install && \
  cp -f src/redis-sentinel /usr/local/bin && \
  mkdir -p /etc/redis && \
  cp -f *.conf /etc/redis && \
  rm -rf /tmp/redis-stable* && \
  sed -i 's/^\(bind .*\)$/# \1/' /etc/redis/redis.conf && \
  sed -i 's/^\(daemonize .*\)$/# \1/' /etc/redis/redis.conf && \
  sed -i 's/^\(dir .*\)$/# \1\ndir \/data/' /etc/redis/redis.conf && \
  sed -i 's/^\(logfile .*\)$/# \1/' /etc/redis/redis.conf

# Install PHP-CS-FIXER
RUN wget https://cs.symfony.com/download/php-cs-fixer-v2.phar -O php-cs-fixer
RUN chmod a+x php-cs-fixer
RUN mv php-cs-fixer /usr/local/bin/php-cs-fixer

# Install NODE 14 and Yarn
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt -y install nodejs
RUN curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt update && apt install yarn


RUN mkdir -p /appdata/www

# Config XDEBUG
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

WORKDIR /appdata/www
