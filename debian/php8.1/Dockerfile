FROM php:8.1.1-fpm-buster

MAINTAINER xyu

# redis version
ENV PHPREDIS_VERSION 5.3.5

# swoole version
ENV SWOOLE_VERSION 4.8.4

# composer version
ENV COMPOSER_VERSION 2.1.12

# 设置时区
ENV TZ=Asia/Shanghai

RUN set -eux;\
        mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
        echo "deb http://ftp.de.debian.org/debian/ buster main non-free contrib" > /etc/apt/sources.list && \
          echo "deb-src http://ftp.de.debian.org/debian/ buster main non-free contrib" >> /etc/apt/sources.list && \
          echo "deb http://security.debian.org/ buster/updates main contrib non-free" >> /etc/apt/sources.list && \
          echo "deb-src http://security.debian.org/ buster/updates main contrib non-free" >> /etc/apt/sources.list && \
         apt-get -qq update && apt-get -qqy upgrade && apt-get -yqq install wget && \
# timezone
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
# source extract
docker-php-source extract && \
# package build
mkdir /packages && \

## 手动下载最新稳定版本 openssl
cd /packages && \
wget https://github.com/openssl/openssl/archive/refs/tags/OpenSSL_1_1_1m.tar.gz && \
tar -xzf  OpenSSL_1_1_1m.tar.gz && \
cd openssl-OpenSSL_1_1_1m && \
./config && \
make && make install && \
cd ../ && \

# 手动下载最新稳定版本 curl
#cd /packages && \
#wget https://github.com/curl/curl/releases/download/curl-7_79_1/curl-7.79.1.tar.gz && \
#tar -xzf  curl-7.79.1.tar.gz && \
#cd curl-7.79.1 && \
#./configure --without-ssl && \
#make && make install && \
#cd ../ && \

# Install some basic tools needed for deployment
  apt-get -yqq install \
  apt-utils \
  build-essential \
  debconf-utils \
  debconf \
  default-mysql-client \
  locales \
  curl \
  unzip \
  patch \
  rsync \
  vim \
  nano \
  openssh-client \
  git \
  bash-completion \
  locales \
  libjpeg-turbo-progs libjpeg-progs \
  pngcrush optipng \
  --no-install-recommends pkg-config build-essential libsodium-dev libedit-dev \
  bzip2 unzip libbz2-dev libcurl4-openssl-dev libonig-dev libmagickwand-dev libicu-dev libldap2-dev libxslt-dev \
  autoconf bison re2c libxml2-dev libsqlite3-dev \
  libjpeg-dev libpng-dev libjpeg62-turbo-dev zlib1g-dev libmcrypt-dev  libreadline6-dev libfreetype6-dev \
  libtidy-dev libtool valgrind libssl-dev libzip-dev libwebp-dev libxpm-dev && \

# extension
docker-php-ext-install gd  && \
docker-php-ext-enable gd  && \
docker-php-ext-install pdo_mysql  && \
docker-php-ext-enable pdo_mysql  && \
docker-php-ext-install sockets  && \
docker-php-ext-enable sockets  && \
docker-php-ext-install opcache  && \
docker-php-ext-enable opcache  && \
docker-php-ext-install mysqli  && \
docker-php-ext-enable mysqli  && \
docker-php-ext-install pcntl  && \
docker-php-ext-enable pcntl  && \
docker-php-ext-install bcmath  && \
docker-php-ext-enable bcmath  && \
docker-php-ext-install bz2  && \
docker-php-ext-enable bz2  && \
#docker-php-ext-install xmlrpc  && \
#docker-php-ext-enable xmlrpc  && \
docker-php-ext-install gettext  && \
docker-php-ext-enable gettext  && \
docker-php-ext-install intl  && \
docker-php-ext-enable intl  && \
docker-php-ext-install zip  && \
docker-php-ext-enable zip  && \

# Redis 扩展下载 pecl本地安装 开启扩展
cd /packages && \
wget http://pecl.php.net/get/redis-${PHPREDIS_VERSION}.tgz && \
tar -xzvf redis-${PHPREDIS_VERSION}.tgz && \
cd redis-${PHPREDIS_VERSION} && \
    /usr/local/bin/phpize && \
    ./configure --with-php-config=/usr/local/bin/php-config && \
    make && make install && \
    cd ../ && \
    docker-php-ext-enable redis && \

# Swoole 扩展安装 开启扩展
cd /packages && \
    wget https://github.com/swoole/swoole-src/archive/refs/tags/v${SWOOLE_VERSION}.tar.gz && \
    tar -xzvf v${SWOOLE_VERSION}.tar.gz && \
    cd swoole-src-${SWOOLE_VERSION} && \
    /usr/local/bin/phpize && \
    ./configure --enable-swoole-curl --enable-sockets --enable-swoole-json --enable-mysqlnd --enable-openssl --enable-http2 && \
    make && make install && \
    cd ../ && \
    docker-php-ext-enable swoole && \

# confiugure
cd /usr/local/etc/php && \
cp php.ini-production php.ini && \
sed -i 's/display_errors\s*=.*/display_errors = Off/' php.ini && \
sed -i 's/error_reporting\s*=.*/error_reporting = E_ALL \& ~E_NOTICE \& ~E_STRICT \& ~E_DEPRECATED/' php.ini && \
sed -i 's/;error_log\s*=\s*php_errors\.log/error_log = \/var\/log\/php_errors.log/' php.ini && \
sed -i 's/;date\.timezone\s*=.*/date.timezone = Asia\/Shanghai/' php.ini && \
sed -i 's/display_errors\s*=.*/swoole.use_shortname=Off/' php.ini && \

# package delete
rm -rf /packages && \
# composer
curl -sfL https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer && \
chmod +x /usr/bin/composer                                                                     && \
# source delete
docker-php-source delete && \
# Cleanup some things
apt-get -q autoclean && \
rm -rf /var/lib/apt/lists/*

WORKDIR /data

EXPOSE 9001