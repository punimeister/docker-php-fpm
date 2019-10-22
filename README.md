# Please use only during development

# Supported tags and respective `Dockerfile` links

- [`7.3`, `latest` (7.3/Dockerfile)](https://github.com/punimeister/docker-php-fpm/blob/master/7.3/Dockerfile)
- [`7.2` (7.2/Dockerfile)](https://github.com/punimeister/docker-php-fpm/blob/master/7.2/Dockerfile)
- [`7.1` (7.1/Dockerfile)](https://github.com/punimeister/docker-php-fpm/blob/master/7.1/Dockerfile)

## Environment Variables

### `REMOTE_DEBUG` (default = 0)

When set to 1 enable to remote debug (Xdebug)

## Configuration files (in docker container)

- Nginx config ... `/etc/nginx/conf.d/default.conf` in nginx
- php.ini ... `/usr/local/etc/php/php.ini` in php-fpm
- xdebug.ini ... `/usr/local/etc/php/conf.d/xdebug.ini` in php-fpm
- rootCA.pem ... `/root/.local/share/mkcert/rootCA.pem` in nginx
- rootCA-key.pem ... `/root/.local/share/mkcert/rootCA-key.pem` in nginx

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

```yaml
version: '3'

services:

  nginx:
    build: ./nginx
    restart: on-failure
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./certs:/root/.local/share/mkcert
      - ./web:/var/www/html
    depends_on:
      - php-fpm

  php-fpm:
    build: punimeister/php-fpm:latest
    restart: on-failure
    environment:
      REMOTE_DEBUG: 0
    volumes:
      - ./web:/var/www/html
```

### Dockerfile

```
FROM nginx:latest

RUN apt-get update \
 && apt-get install -y --no-install-recommends --no-install-suggests wget \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/bin
RUN wget --no-check-certificate -O mkcert https://github.com/FiloSottile/mkcert/releases/download/v1.4.0/mkcert-v1.4.0-linux-amd64 \
 && chmod +x mkcert \
 && mkdir -m 755 /certs

COPY site.conf /etc/nginx/conf.d/default.conf

WORKDIR /

CMD sh -c " \
      mkcert -install \
      && mkcert -cert-file /certs/docker.pem -key-file /certs/docker-key.pem \
         localhost \
         127.0.0.1 \
         192.168.99.100 \
      && nginx -g 'daemon off;' \
    "
```

### site.conf

```
server {
  listen 80;
  charset utf-8;

  root /var/www/html/public;

  index index.php index.html;

  add_header X-Frame-Options "SAMEORIGIN";
  add_header X-XSS-Protection "1; mode=block";
  add_header X-Content-Type-Options "nosniff";
  server_tokens off;

  error_log  /var/log/nginx/error.log;
  access_log /var/log/nginx/access.log;

  location ~ \.php$ {
    try_files $uri /dev/null =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass php-fpm:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_hide_header X-Powered-By;
  }
}

server {
  listen 443 ssl;
  charset utf-8;

  root /var/www/html/public;

  index index.php index.html;

  ssl_certificate      /certs/docker.pem;
  ssl_certificate_key  /certs/docker-key.pem;

  add_header X-Frame-Options "SAMEORIGIN";
  add_header X-XSS-Protection "1; mode=block";
  add_header X-Content-Type-Options "nosniff";
  server_tokens off;

  error_log  /var/log/nginx/error.log;
  access_log /var/log/nginx/access.log;

  location ~ \.php$ {
    try_files $uri /dev/null =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass php-fpm:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_hide_header X-Powered-By;
  }
}
```
