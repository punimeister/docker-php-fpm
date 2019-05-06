FROM php:5.6-fpm

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
      libmcrypt-dev \
      libmemcachedutil2 \
      libpng-dev \
      libpq-dev \
      libxml2-dev \
    " \
 && apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends $buildDeps $runtimeDeps \
 && docker-php-ext-install bcmath bz2 calendar iconv intl mbstring mcrypt mysql mysqli opcache pdo_mysql pdo_pgsql pgsql soap zip \
 && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
 && docker-php-ext-install gd \
 && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
 && docker-php-ext-install ldap \
 && docker-php-ext-install exif \
 && pecl install memcached-2.2.0 redis \
 && docker-php-ext-enable memcached.so redis.so \
 && apt-get purge -y --auto-remove $buildDeps \
 && rm -rf /var/lib/apt/lists/*

# Install Composer.
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
 && ln -s $(composer config --global home) /root/composer
ENV PATH=$PATH:/root/composer/vendor/bin COMPOSER_ALLOW_SUPERUSER=1

# Install Xdebug.
RUN pecl install xdebug-2.5.5

# Copy ini files.
COPY php.ini /usr/local/etc/php/
COPY xdebug.ini /usr/local/etc/php/conf.d/

# Initial settings.
ENV REMOTE_DEBUG='0' PHP_IDE_CONFIG='serverName=localhost'
WORKDIR /var/www/html

# 1. Rewrite xdebug.ini if needed.
# 2. Starting PHP-FPM
CMD sh -c " \
      if [ "${REMOTE_DEBUG}" = "1" ]; then \
        sed -i 's/\;xdebug\.remote/xdebug\.remote/' /usr/local/etc/php/conf.d/xdebug.ini; \
      fi \
      && php-fpm \
    "
