# Dockerizing application

## installation

install `laravel/horizon`, `laravel/octane`, `spiral/roadrunner`, `predis/predis` and `nunomaduro/larastan`. read each package for installation and configuration steps.
for `predis/predis` set `database.redis.client` to `predis`

then copy all files in this directory into your Laravel project and put variables into your .env file.

``` env
NGINX_IMAGE=app/nginx:latest
APPLICATION_IMAGE_fpm=app/application_fpm:latest
APPLICATION_IMAGE=app/application:latest
QUEUE_WORKER_IMAGE=app/queue_worker:latest
```

## usage

### running with nginx+fpm

`docker-compose up -d`

### running with octane (roadrunner)

`docker-compose -f docker-compose.rr.yml up -d`
