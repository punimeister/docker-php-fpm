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
