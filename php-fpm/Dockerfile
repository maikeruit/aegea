FROM php:7.4-fpm

# Arguments defined in docker-compose.yml
ARG user
ARG uid

RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libonig-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd mbstring pdo_mysql mysqli \
    && docker-php-source delete \
    && rm -rf /var/lib/apt/lists/*

COPY ./docker-php-egeya.ini /usr/local/etc/php/conf.d

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u $uid -d /home/$user $user
RUN mkdir -p /home/$user/.composer; \
    chown -R $user:$user /home/$user

WORKDIR /var/www/html

EXPOSE 9000	

CMD ["php-fpm"]