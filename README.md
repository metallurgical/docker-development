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

WORKDIR /var/www/html

RUN docker-php-ext-install mbstring pdo pdo_mysql \
    && chown -R www-data:www-data /var/www/html \
    && a2enmod rewrite
    
```

#### Explanation

- **FROM**: From which image our image is extending from
- **MAINTAINER**: Our image's maintainer
- **COPY**: Copy current directory(all our laravel's file) into `/var/www/html` folder on where our image container reside
- **COPY**: Copy apache virtual host file to handle our laravel project's request into `/etc/apache2/sites-available/00-default.conf` apache default vhost in our docker container
- **WORKDIR** - Specify our working directory on our docker container
- **RUN**
    - **docker-php-ext-install**: PHP official command/interface to install library required by our laravel project
    - **chown**: Change ownership of html folder
    - **a2enmod**: enable apache modrewrite to enable laravel's htaccess file


### Create vhost.conf

To make our laravel project available when browsing in browser, we need to create default virtual host to handle the request and map the request into directory of where our project is reside, so that apache will know where our project is located. Paste following content:

```
<VirtualHost *:80>
     DocumentRoot /var/www/html/public

     <Directory "/var/www/html/public">
         AllowOverride all
         Require all granted
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
 </VirtualHost>
 ```
 
 To handle multiple laravel project, we can create specific virtual host to each of projects.
 
 ### Build and run the image using docker build
 
Before we can use this environment, we need to build our docker image by running following command inside `~/projects/docker/laravel-app` folder:

```
docker build --file .docker/Dockerfile -t laravel-app .
```

#### Explanation

- **build**: Build our docker image before able to use it
- **--file**: Since our dockerfile reside on different path, we need to specify which dockerfile to use to build the image(default location is where the command of `docker` executed)
- **-t**: Renaming our docker image 
- **.**: Build will run on current folder where `docker` command is executed

Once finish build the image, we can run it to test it out from browser by using following command:

```
docker run <--rm> -p 8080:80 laravel-app
```

#### Explanation

- **run**: Run the following image
- **--rm**: This option will remove our container when we press `ctrl + c` when stopping container instance
- **-p**: Specify which port to listen to and forward the port into container instance. 8080(Our machine's port), 80(Container's port for http)
- **laravel-app**: Our built image

Open web browser and hit `http://localhost:8080`, and you should see our laravel project up and running inside our docker container instance
 
 
 
### Build and run the image using docker compose(docker-compose.yml)
 
As for now, when we make some changes to our working directory, no changes will reflect to our laravel app. As for development environment, this is important and vital. We can make the changes into our working and re-build the image which makes repeatative works. To avoid this use case, let create `docker-compose.yml` file to instruct our image to always listen the changes we made on working directory and straight away reflect to laravel app without re-build the image. Paste following contents: 

```
version: '3'                                    // Recommended version for YAML
services:                                       // Our services that will be run when our image is up and running
    application:    --> 1st service(our laravel app)
        build:                                  // Build information
            context: .                          // Run on current context(current directory)
            dockerfile: ./docker/Dockerfile     // Which dockerfile to look for(since we put inside custom directory)
        image: laravel-app                      // Build the image with name "laravel-app"
        ports:                                  // Specifiy port. Port 9000(from our browser) will forward to port 80(our container)
            - 9000:80
        volumes:                                // This will ensure the container's image always reflect the changes made to working directory
            - .:/var/www/html                   // From .(current directory) bind to `/var/www/html`
        links:                                  // Make networking available to make our laravel-app can communicate with mysql database
            - mysql                             // Name of mysql services(can be naming anything)
        environment:                            // Environment variables that available for us to use to replace the defaut laravel's environment variable(.env)
            DB_HOST: mysql                      // Laravel will use this variable for DB HOST
            DB_DATABASE: laravel_app            // Laravel will use this variable for DB DATABASE
            DB_USERNAME: root                   // Laravel will use this variable for DB USERNAME
            DB_PASSWORD: secret                 // Laravel will use this variable for DB PASSWORD
        
    mysql:       --> 2nd service(our mysql database)
        image: mysql:latest                     // Pull from existing image on docker hub
        ports:                                  // Specifiy port. Port 12345(from our local) will forward to port 3306(our container)
            - 12345:3306
        environment:                            // Environment variables that available for us to use to replace the defaut mysql's environment variable
            MYSQL_DATABASE: laravel_app
            MYSQL_USER: root
```

Once finish, we must build the image again and at the same time run docker compose to up and running the container:

```
$ docker-compose up --build // or docker-compose build && docker-compose up
```

This command will read the `docker-compose.yml`'s file and executed the instruction line by line. From this point, now you're able to browser our laravel-app using `http://localhost:9000`. When the browser detect port 9000, the port forwarding will occur which forward the port 9000 to port 80 of docker container.


### Execute laravel command inside docker container instance

Some of the laravel's command need to be run inside docker container instance itself. Luckily, docker do provide command for us to manage container instance.

Run following command to look for container ID(find "laravel-app"'s name) and copy the container ID:

```
$ docker ps -a
```

Once got the container ID, we can enter into container instance using following command:

```
$ docker exec -it <container-ID> bash
```

Once success, go into our working directory inside container:

```
$ cd /var/www/html
```

and run following command(just an example)

```
$ php artisan migrate
```

This will migrate the existing migrations files into specify database.


### Connect container's database using Sequel Pro(any db client) from our local

```
HOST: 127.0.0.1/localhost   // Default hostname
PORT: 12345                 // This port we specify inside docker-compose.yml file
USER: root                  // This port we specify inside docker-compose.yml file
PASSWORD: secret            // This port we specify inside docker-compose.yml file
```
            
 
 
