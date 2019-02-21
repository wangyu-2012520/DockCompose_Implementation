# DockCompose_Implementation

## entrypoint
Override the default entrypoint.

how to code entrypoint
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
    entrypoint: sh -c "sleep 10 && dotnet WooliesX.Satalia.TransitFileGenerator.Service.dll"
```  

Note: Setting entrypoint both overrides any default entrypoint set on the service’s image with the ENTRYPOINT Dockerfile instruction, and clears out any default command on the image - meaning that if there’s a CMD instruction in the Dockerfile, it is ignored.
