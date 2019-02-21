# DockCompose_Implementation

## entrypoint
docker compose entrypoint will override the default entrypoint in docker file.
simple way to code entrypoint
```
    entrypoint: /code/entrypoint.sh
```
The entrypoint can also be a list, in a manner similar to dockerfile:
```
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```
In my code, it looks like
```
    entrypoint: sh -c "sleep 10 && dotnet X.Service.dll"
```  

**Note: Setting entrypoint both overrides any default entrypoint set on the service’s image with the ENTRYPOINT Dockerfile instruction, and clears out any default command on the image - meaning that if there’s a CMD instruction in the Dockerfile, it is ignored**.

## entrypoint vs cmd

entrypoint - Entrypoint sets the command and parameters that will be executed first when a container is run.

cmd - The main purpose of a CMD (Dockerfiles) / command (Docker Compose files) is to provide defaults when executing a container (it is more like an argeument passing to executing file). These will be executed after the entrypoint.
For example, if you ran docker run <image>, then the commands and parameters specified by CMD / command in your Dockerfiles would be executed.
