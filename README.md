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
```
version: '3'
services:
  mock-server:
    build:
      context: ../../WooliesX.Satalia.MockServer
    command: ["micro-service", "./resources/mockPostAuth.json", "./resources/mockGetAllStores.json", "./resources/mockPlans_cutoffTime_before.json ./resources/mockPlans_cutoffTime_after.json"]
    ports:
      - "5002:5002"
  mssql:
    build:
      context: ../../WooliesX.TransitFile.Database
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=G00dPassw0rd
    ports:
      - "1433:1433"
    restart: always 
  store-retriever:
    image: "wowdevcontainers.azurecr.io/store-retriever"
    ports:
      - "5003:5002"
    depends_on:
      - "mock-server"
      - "mssql"
    env_file:
      - ../dockersupport/env-variables.env
  transit-generator:
    image: "wowdevcontainers.azurecr.io/transit-generator"
    ports:
      - "5004:5002"
    depends_on:
      - "mssql"
      - "mock-server"
    env_file:
      - ../dockersupport/env-variables.env
    environment:
      - SFG_RouteFinalise_CutoffTime=23:59
    entrypoint: sh -c "sleep 10 && dotnet WooliesX.Satalia.TransitFileGenerator.Service.dll"
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
