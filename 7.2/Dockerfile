FROM php:7.2-fpm

# Install PHP extensions and PECL modules.
RUN buildDeps=" \
      default-libmysqlclient-dev \
      libbz2-dev \
      libmemcached-dev \
      libsasl2-dev \
    " \
    runtimeDeps=" \
      curl \
      git \
      libfreetype6-dev \
      libicu-dev \
      libjpeg-dev \
      libldap2-dev \
      libmemcachedutil2 \
      libpng-dev \
      libpq-dev \
      libxml2-dev \
      unzip \
    " \
 && apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends $buildDeps $runtimeDeps \
 && docker-php-ext-install bcmath bz2 calendar iconv intl mbstring mysqli opcache pdo_mysql pdo_pgsql pgsql soap zip \
 && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
 && docker-php-ext-install gd \
 && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
 && docker-php-ext-install ldap \
 && docker-php-ext-install exif \
 && pecl install memcached redis \
 && docker-php-ext-enable memcached.so redis.so \
 && apt-get purge -y --auto-remove $buildDeps \
 && rm -rf /var/lib/apt/lists/*

# Install Composer.
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
 && ln -s $(composer config --global home) /root/composer
ENV PATH=$PATH:/root/composer/vendor/bin COMPOSER_ALLOW_SUPERUSER=1

# Install Xdebug.
RUN pecl install xdebug-2.7.2

# Copy ini files.
COPY php.ini /usr/local/etc/php/
COPY xdebug.ini /usr/local/etc/php/conf.d/

# Declare default environment variables.
ENV PHP_IDE_CONFIG 'serverName=localhost'
ENV REMOTE_DEBUG 0

WORKDIR /var/www/html

# 1. Rewrite xdebug.ini if necesarry.
# 2. Start PHP-FPM.
CMD sh -c " \
      if [ "${REMOTE_DEBUG}" = "1" ]; then \
        sed -i 's/\;xdebug\.remote/xdebug\.remote/' /usr/local/etc/php/conf.d/xdebug.ini; \
      fi \
      && php-fpm \
    "
