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

cmd - The main purpose of a CMD (Dockerfiles) / command (Docker Compose files) is to provide defaults when executing a container (it is more like an argeument passing to executing container). These will be executed after the entrypoint.
For example, if you ran docker run <image>, then the commands and parameters specified by CMD / command in your Dockerfiles would be executed.
    
NOTE: There can only be one CMD instruction in a Dockerfile. If you list more than one CMD, then only the last CMD will take effect.

## example 
here is the example of one docker-compose file. The following docker-compose file contains 5 docker containers, some of them are images, some of them will build images and run container. 

There is dependency on each container, when docker-compose file is fire /up, all docker containers are trigger /up in order. For example, the key word 'depends_on' used on image 'transit-generator' only means that image 'transit-generator' will run straight after both images 'mssql' and 'mock-server' are up, but it does not guarantee that iamge 'transit-generator' will wait until images 'mssql' and 'mock-server' are fully ready. 

This example is quite good example, because image 'mssql' will take up to 7-8 seconds to restore everything, get ready and 1-2 seconds to import init data, if there is no manual interaction the image 'transit-generator' will try to read data from database and tables and get failed (because image 'transit-generator' is quick, it only takes it 3 seconds to reach the lines of reading database, but the database is not ready yet). In this case, we need to put image 'transit-generator' on hold and let it sleep 10s before it fire up.
```
version: '3'
services:
  mock-server:
    build:
      context: ../../Woo.Sat.MockServer
    command: ["micro-service", "./resources/mockPostAuth.json", "./resources/mockGetAllStores.json", "./resources/mockPlans_cutoffTime_before.json ./resources/mockPlans_cutoffTime_after.json"]
    ports:
      - "5002:5002"
  mssql:
    build:
      context: ../../Woo.TransitFile.Database
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=G00dPassw0rd
    ports:
      - "1433:1433"
    restart: always 
  store-retriever:
    image: "store-retriever"
    ports:
      - "5003:5002"
    depends_on:
      - "mock-server"
      - "mssql"
    env_file:
      - ../dockersupport/env-variables.env
  transit-generator:
    image: "transit-generator"
    ports:
      - "5004:5002"
    depends_on:
      - "mssql"
      - "mock-server"
    env_file:
      - ../dockersupport/env-variables.env
    environment:
      - SFG_RouteFinalise_CutoffTime=23:59
    entrypoint: sh -c "sleep 10 && dotnet Woo.Sat.TransitFileGenerator.Service.dll"
  integration-test:
    build:
      context: ../
    command: --filter BeforeCutoffTimeRoutesShould
    depends_on:
      - "mock-server"
      - "mssql"
      - "store-retriever"
      - "transit-generator"
```
