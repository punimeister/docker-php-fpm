# Please use only during development

# Supported tags and respective `Dockerfile` links

- [`7.3`, `latest` (7.3/Dockerfile)](https://github.com/punimeister/docker-php-fpm/blob/master/7.3/Dockerfile)
- [`7.2` (7.2/Dockerfile)](https://github.com/punimeister/docker-php-fpm/blob/master/7.2/Dockerfile)
- [`7.1` (7.1/Dockerfile)](https://github.com/punimeister/docker-php-fpm/blob/master/7.1/Dockerfile)
- [`5.6` (5.6/Dockerfile)](https://github.com/punimeister/docker-php-fpm/blob/master/5.6/Dockerfile)

## Environment Variables

### `REMOTE_DEBUG` (default = 0)

When set to 1 enable to remote debug (Xdebug)

## Configuration files (in docker container)

<!-- - Nginx HTTP config ... `/etc/apache2/sites-enabled/000-default.conf` in nginx -->
<!-- - Nginx HTTPS config ... `/etc/apache2/sites-enabled/000-ssl.conf` in nginx -->
- php.ini ... `/usr/local/etc/php/php.ini` in php-fpm
- xdebug.ini ... `/usr/local/etc/php/conf.d/xdebug.ini` in php-fpm
<!-- - rootCA.pem ... `/root/.local/share/mkcert/rootCA.pem` in nginx -->
<!-- - rootCA-key.pem ... `/root/.local/share/mkcert/rootCA-key.pem` in nginx -->

## Example

### Directory structure

```
.
├── certs
├── docker-compose.yml
├── nginx
│   ├── Dockerfile
│   └── site.conf
└── web
    └── public
        └── index.php
```

### docker-compose.yml

```
version: '3'

services:

  nginx:
    build: './nginx'
    restart: 'on-failure'
    ports:
      - '80:80'
    volumes:
      - './web:/var/www/html'

  php-fpm:
    build: 'punimeister/php-fpm:latest'
    restart: 'on-failure'
    environment:
      REMOTE_DEBUG: '0'
    volumes:
      - './web:/var/www/html'
```
