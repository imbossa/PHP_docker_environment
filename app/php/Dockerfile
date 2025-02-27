ARG PHP_VERSION
FROM php:${PHP_VERSION}

LABEL maintainer="bossa"
ENV GIT_SSL_NO_VERIFY true

# replace sources.list
#RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
#     && sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list

# init
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    curl \
    wget \
    zip \
    unzip \
    vim \
    vim-common \
    git \
    locales \
    procps \
    openssl \
    libz-dev \
    libpq-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libssl-dev \
    libmcrypt-dev \
    libmemcached-dev \
    libtool \
    libicu-dev \
    libxml2 \
    libxml2-dev \
    libgmp3-dev \
    libevent-dev \
    build-essential \
    libzstd-dev \
    autoconf \
    libzip-dev \
    zlib1g-dev \
    libldap2-dev \
    libbz2-dev \
    libjpeg62-turbo-dev \
    libtidy-dev \
    libsqlite3-dev \
    libsqlite3-0 \
    tmux \
    keychain \
    net-tools \
    python3 \
    supervisor \
    cron \
    iputils-ping \
    telnet \
    && rm -rf /var/lib/apt/lists/*

RUN echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen

ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

RUN pecl channel-update pecl.php.net \
        && docker-php-ext-configure gd \
        --with-jpeg=/usr/lib \
        --with-freetype=/usr/include/freetype2

RUN docker-php-ext-install -j$(nproc) dba bz2 calendar zip tidy bcmath mysqli pdo_mysql gd pcntl intl soap gmp sockets

# install php extentions
RUN pecl install redis \
    && docker-php-ext-enable redis
RUN pecl install memcached \
    && docker-php-ext-enable memcached
RUN pecl install mongodb \
    && docker-php-ext-enable mongodb
RUN pecl install msgpack \
    && docker-php-ext-enable msgpack
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug
RUN pecl install inotify \
    && docker-php-ext-enable inotify
RUN pecl install protobuf \
    && docker-php-ext-enable protobuf
RUN pecl install event \
    && docker-php-ext-enable event
RUN pecl install grpc \
    && docker-php-ext-enable grpc
RUN pecl clear-cache \
  && echo "[finish install php]"

RUN rm /usr/local/etc/php/conf.d/docker-php-ext-event.ini
RUN docker-php-ext-enable --ini-name z999-docker-php-ext-event.ini event

# copy ssh key
COPY secure/ssh/ /root/.ssh/
COPY entrypoint.sh /usr/local/bin/
COPY supervisor/ /etc/supervisor/

RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && service cron stop \
    && service cron start \
    && git config --global http.postBuffer 2000000000 \
    && git config --global http.sslVerify false \
    && mkdir -p /var/log/www \
    && chown -R www-data:www-data /var/log/www && chown -R www-data:www-data /var/www \
    && chmod +x /usr/local/bin/entrypoint.sh \
    && chmod -R 0600 /root/.ssh

WORKDIR /var/www/

ENTRYPOINT ["entrypoint.sh"]