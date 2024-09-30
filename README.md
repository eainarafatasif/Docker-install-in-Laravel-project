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
├── laravel-project----------------------------------- #main project folder
├── mysql ---------------------------------------------#create folder
├── nginx ---------------------------------------------#create folder
│    ├── conf.d-- -------------------------------------#create folder
│         └── default.conf 
├── php -----------------------------------------------#create folder
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
