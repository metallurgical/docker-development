# docker-development
Set of instructions to use docker as local environment without cluttering your local development with diversity of library versioning etc.

For brevity, this instruction will use Laravel as PHP Framework, MYSQL and APACHE which connecting to each others. 

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

### Installations
