# STAGE 1: Nodejs build (js/css assets) =======================================
FROM node:14.7.0 as nodejs

WORKDIR /tmp

# Work with package files first to take advantage of cache
COPY package.json package-lock.json ./
RUN npm install

# Same thing here, copy required assets only to leverage cache
COPY resources ./resources
COPY webpack.mix.js ./webpack.mix.js
COPY .babelrc.json ./.babelrc.json

RUN npm run production


# STAGE 2: PHP + Laravel ======================================================
FROM php:7.4.9-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

COPY . .

# Copy production-ready assets from stage 1
COPY --from=nodejs /tmp/public ./public

RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html

RUN composer install --no-dev --optimize-autoloader

# Set the application key & clear cache
RUN php artisan key:generate \
    && php artisan cache:clear \
    && php artisan config:clear \
    && php artisan route:clear \
    && php artisan config:cache \
#    && php artisan route:cache \ # will not work if using routes w/ closures (use controllers)
    && php artisan view:cache
    # or can just use "php artisan optimize" which clears and recaches everything

# Other steps (I prefer to do manually)
## Run migrations
## Run env-specific db seeders

# PHP-FPM default listening port
EXPOSE 9000

# We run this and this is what nginx is communicating with
CMD ["php-fpm"]
