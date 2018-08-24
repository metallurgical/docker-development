# docker-development
Set of instructions to use docker as local environment without cluttering your local development with diversity of library versioning etc.

For the sake of brevity, this instruction will use Laravel as PHP Framework, MYSQL and APACHE which connecting to each others. 

### Requirements
1) Docker, yeah docker only

### Final Folders Structures
The only files that important to pay attention(*)

```
|-- .docker
    |-- Dockerfile        // Having instruction to build our image *
    |-- Vhost.conf .      // For default virtual host copying into docker container's image *
|-- .env
|-- .env.example
|-- .gitignore
|-- app
|-- artisan
|-- bootstrap
|-- composer.json
|-- composer.lock
|-- config
|-- database
|-- docker-compose.yml  // Complete task to run when running up docker container *
|-- package.json
|-- phpunit.xml
|-- public
|-- readme.md
|-- resources
|-- routes
|-- server.php
|-- storage
|-- tests
|-- vendor
|-- webpack.mix.js
```

From this point of view, i separated out `docker` files from main project's root and putting it into their own folder `.docker` except for `docker-compose.yml`

### Create Laravel project

Before we can start development, we need to create `laravel` project inside our local environment before we copy the data into docker container. For this example, i'll create under `~/projects/dockers/<project-name>` folder:

```
composer create-project --prefer-dist laravel/laravel laravel-app
```

Make sure you have `composer` install in your local machine.


### Create Dockerfile

This file having sets of instructions to build our main image that extending from existing docker hub's image. For this example i'll use the PHP 7.2-apache image from [PHP official](https://hub.docker.com/_/php/) docker hub. Put following contents: 

```
FROM PHP:7.2-apache
MAINTAINER Norlihazmey Ghazali

COPY . /var/www/html
COPY .docker/vhost.conf /etc/apache2/sites-available/00-default.conf

RUN docker-php-ext-install mbstring pdo pdo_mysql \
    && chown -R www-data:www-data /var/www/html \
    && a2enmod rewrite
    
```




