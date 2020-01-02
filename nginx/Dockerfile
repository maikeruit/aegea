FROM debian:10-slim

ARG PHP_UPSTREAM_CONTAINER=php-fpm
ARG PHP_UPSTREAM_PORT=9000

RUN apt-get update && apt-get install -y \
	nginx \
	&& rm -rf /var/lib/apt/lists/*

WORKDIR /etc/nginx

RUN echo "upstream php-upstream { server ${PHP_UPSTREAM_CONTAINER}:${PHP_UPSTREAM_PORT}; }" > /etc/nginx/conf.d/upstream.conf \
	&& rm -rf /etc/nginx/sites-enabled/*

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]