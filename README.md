# Docker-install-in-Laravel-project


To deploy a Laravel application on Docker using MySQL, Nginx, and SSL, you must create a Docker setup using Docker Compose. Below is a step-by-step guide:

Prerequisites:
Docker installed on your system.
Docker Compose installed.
A basic Laravel project setup.

Example Project Structure
Here’s an example of how your project might be structured for clarity:

```bash
my_project/
├── laravel-project----------------------------------- >main project folder
├── mysql --------------------------------------------->create folder
├── nginx --------------------------------------------->create folder
│    ├── conf.d-- ------------------------------------->create folder
│         └── default.conf 
├── php ----------------------------------------------->create folder
│   └── www.conf   
├── Dockerfile
├── nginx.dockerfile
├── docker-compose.yml


 ```

Step 1: Create a docker-compose.yml file
In your Laravel project root directory, create a docker-compose.yml file to define the services (Laravel, MySQL, Nginx, etc.) and their configurations.

```bash
version: '3.8'

networks: 
  laravel:
    name: laravel

services: 
  nginx:
    build:
      context: .
      dockerfile: nginx.dockerfile
    container_name: blog_nginx
    tty: true
    depends_on:
      - php
      - mysql
    ports: 
      - 80:80
      - 443:443
    volumes:
      - ./relaxarc-travel-main:/var/www/html  
    networks:
      - laravel
      
  php:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: blog_php
    restart: unless-stopped
    tty: true
    volumes:
      - ./relaxarc-travel-main:/var/www/html
    networks:
      - laravel

  mysql:
    image: mysql:latest
    container_name: blog_mysql
    ports:
      - "3306:3306"
    volumes:
      - ./mysql:/var/lib/mysql  
    environment:
      MYSQL_DATABASE: docker_laravel
      MYSQL_USER: laravel
      MYSQL_PASSWORD: secret
      MYSQL_ROOT_PASSWORD: secret
    networks:
      - laravel

  phpmyadmin:
    depends_on:
      - mysql
    image: phpmyadmin:latest
    container_name: phpmyadmin_docker
    restart: unless-stopped
    ports:
      - "1002:80"
    environment:
      PMA_HOST: mysql
    networks:
      - laravel


```
Step 2: Create a Dockerfile for PHP (Laravel)
In the same directory, create a Dockerfile for the Laravel PHP container:


```bash

# Use PHP 8.3 with FPM on Alpine
FROM php:8.1.0-fpm-alpine

# Add your PHP configuration
ADD ./php/www.conf /usr/local/etc/php-fpm.d/www.conf

# Add Laravel user and group
RUN addgroup -g 1000 laravel && adduser -G laravel -g laravel -s /bin/sh -D laravel

# Create the application directory
RUN mkdir -p /var/www/html

WORKDIR /var/www/html

# Install required packages for Composer, git, unzip, curl, Node.js, and npm
RUN apk add --no-cache curl git unzip nodejs npm

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install PHP extensions (pdo_mysql, zip, gd, redis)
RUN docker-php-ext-install pdo pdo_mysql \
    && apk --no-cache add libzip-dev zlib-dev libpng-dev libjpeg-turbo-dev freetype-dev \
    && docker-php-ext-configure gd --with-jpeg --with-freetype \
    && docker-php-ext-install zip gd

# Install Redis extension
RUN apk --no-cache add --virtual .build-deps $PHPIZE_DEPS \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && apk del .build-deps

# Ensure Laravel has ownership of the application files
RUN chown -R laravel:laravel /var/www/html

# Switch to the Laravel user
USER laravel

# Install npm dependencies (if applicable)
# Copy the package.json and package-lock.json before running npm install
COPY --chown=laravel:laravel package*.json ./

# Debugging: List files in the working directory
RUN ls -la /var/www/html

# Install npm packages
RUN npm install --verbose || { echo "NPM install failed"; exit 1; } <--------------------(if Needs)

# Copy the rest of the application files
COPY --chown=laravel:laravel . .

# Expose port 9000 for PHP-FPM
EXPOSE 9000

# Start the PHP-FPM server
CMD ["php-fpm"]

```

Step 3: Create Nginx Configuration
Create an Nginx configuration file under nginx/nginx.conf to define how Nginx interacts with Laravel.

```bash

server {
    listen 80;
    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/html/public;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}

```


