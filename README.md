# ANISRA MKTG POS and Inventory System

<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

## About Laravel

Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects, such as:

- [Simple, fast routing engine](https://laravel.com/docs/routing).
- [Powerful dependency injection container](https://laravel.com/docs/container).
- Multiple back-ends for [session](https://laravel.com/docs/session) and [cache](https://laravel.com/docs/cache) storage.
- Expressive, intuitive [database ORM](https://laravel.com/docs/eloquent).
- Database agnostic [schema migrations](https://laravel.com/docs/migrations).
- [Robust background job processing](https://laravel.com/docs/queues).
- [Real-time event broadcasting](https://laravel.com/docs/broadcasting).

Laravel is accessible, powerful, and provides tools required for large, robust applications.

## Learning Laravel

Laravel has the most extensive and thorough [documentation](https://laravel.com/docs) and video tutorial library of all modern web application frameworks, making it a breeze to get started with the framework.

If you don't feel like reading, [Laracasts](https://laracasts.com) can help. Laracasts contains over 1500 video tutorials on a range of topics including Laravel, modern PHP, unit testing, and JavaScript. Boost your skills by digging into our comprehensive video library.

## Laravel Sponsors

We would like to extend our thanks to the following sponsors for funding Laravel development. If you are interested in becoming a sponsor, please visit the Laravel [Patreon page](https://patreon.com/taylorotwell).

### Premium Partners

- **[Vehikl](https://vehikl.com/)**
- **[Tighten Co.](https://tighten.co)**
- **[Kirschbaum Development Group](https://kirschbaumdevelopment.com)**
- **[64 Robots](https://64robots.com)**
- **[Cubet Techno Labs](https://cubettech.com)**
- **[Cyber-Duck](https://cyber-duck.co.uk)**
- **[Many](https://www.many.co.uk)**
- **[Webdock, Fast VPS Hosting](https://www.webdock.io/en)**
- **[DevSquad](https://devsquad.com)**
- **[Curotec](https://www.curotec.com/services/technologies/laravel/)**
- **[OP.GG](https://op.gg)**

## Contributing

Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the [Laravel documentation](https://laravel.com/docs/contributions).

## Code of Conduct

In order to ensure that the Laravel community is welcoming to all, please review and abide by the [Code of Conduct](https://laravel.com/docs/contributions#code-of-conduct).

## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via [taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).

## Docker Implementation

This section documents how team members can implement docker container to this application if not already installed. 

```
Note: You need to have docker and docker-compose installed before you can proceed with the following steps.
```

### Steps in applying docker

```
Note: Make sure your application already contains `.env` file before you proceed with the next steps. Some of these variables here will be edited later.
```

#### Step 1 - Setup Dockerfile

Create a file named `Dockerfile` in the root directory and copy the content below. You can, however, modify the Dockerfile to suit your personal preference.

```
FROM php:7.4-fpm

# Arguments defined in docker-compose.yml
ARG user
ARG uid

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libwebp-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libxpm-dev \
    zip \
    unzip

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Setup GD extensions
RUN docker-php-ext-configure gd --with-jpeg --with-freetype
RUN docker-php-ext-install gd

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u $uid -d /home/$user $user
RUN mkdir -p /home/$user/.composer && \
    chown -R $user:$user /home/$user

# Set working directory
WORKDIR /var/www

USER $user
```

#### Step 2 - Setup docker-compose.yml

Create another file inside the root directory of the app and name it `docker-compose.yml`. Copy the content of the file below. You can also edit the file according to your desired preferences. 

```
version: "3.7"
services:
  app:
    build:
      args:
        user: xdonie11
        uid: 1000
      context: ./
      dockerfile: Dockerfile
    image: anisra
    container_name: anisra-app
    restart: unless-stopped
    working_dir: /var/www/
    volumes:
      - ./:/var/www
    networks:
      - anisra

  db:
    platform: linux/x86_64
    image: mysql:5.7
    container_name: anisra-db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_USER: ${DB_USERNAME}
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    ports:
       - "3356:3306"
    volumes:
      - ./docker-compose/mysql:/docker-entrypoint-initdb.d
    networks:
      - anisra

  nginx:
    image: nginx:alpine
    container_name: anisra-nginx
    restart: unless-stopped
    ports:
      - 8000:80
    volumes:
      - ./:/var/www
      - ./docker-compose/nginx:/etc/nginx/conf.d/
    networks:
      - anisra

networks:
  anisra:
    driver: bridge
```

Important things to remember: 
The db service contains a container_name key. This is the value you should place into the `DB_HOST` value in `.env` file. 
The volumes inside the db service and nginx both uses the path `./docker-compose/` please take note of this as you will be creating a new path in the next step. 

#### Step 3 - Create the paths (directories) for the volumes

Create a folder named `docker-compose` inside the root directory of the app and then `cd` into the docker-compose directory. Create two folders named `nginx` and `mysql` respectively.

#### Step 4 - Create the app.conf file

Create the nginx conf file named `app.conf` inside the path `/docker-compose/nginx` directory. Copy the content as shown below. 

```
server {
    listen 80;
    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/public;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
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

You are now ready to run the docker image and container and start the development without the need to install any web services. Follow the next steps to run the docker container using docker-compose

#### Step 5 - Run docker-compose

Follow these commands in your terminal (this can run from either macOS terminal, wsl 2, or linux cli). 

First,  lets build the app.

```
docker-compose build app
```

Then we run the containers. 

```
docker-compose up -d
```

We can check if the container ran properly without any issues by this command. 

```
docker-compose ps
```

We can also check the files with this command. 

```
docker-compose exec app ls -l
```

Once we are already sure about the files that are already in our container, and our containers did not encounter any issues, we can proceed to install our laravel application inside the container by running the following command in the terminal. 

```
docker-compose exec app <laravel syntax>
```

Examples: 

```
docker-compose exec app composer install
docker-compose exec app php artisan key:generate
docker-compose exec app php artisan migrate
docker-compose exec app php artisan db:seed
```

The example above (in order) runs the basic composer command from our terminal through the docker container. You can now start development and have fun coding!!!

Notes: 
If you encounter error while using docker-compose in wsl2, close the terminal and open a new one. This is known bug and I could not find the reference for this bug from github as of this writing. I will update this once I get the link of this reported bug from github once I'm able to find them. 
