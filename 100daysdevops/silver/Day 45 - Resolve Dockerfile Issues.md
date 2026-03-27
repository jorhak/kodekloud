```
The Nautilus DevOps team is working to create new images per requirements shared by the development team. One of the team members is working to create a `Dockerfile` on `App Server 2` in `Stratos DC`. While working on it she ran into issues in which the docker build is failing and displaying errors. Look into the issue and fix it to build an image as per details mentioned below:

  

a. The `Dockerfile` is placed on `App Server 2` under `/opt/docker` directory.  
  

b. Fix the issues with this file and make sure it is able to build the image.  
  

c. Do not change base image, any other valid configuration within Dockerfile, or any of the data been used — for example, index.html.  
  

`Note:` Please note that once you click on `FINISH` button all the existing containers will be destroyed and new image will be built from your `Dockerfile`.
```

## Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Ejecutar el Dockerfile para ver el error
```
cd /opt/docker
docker buildx build --platform linux/amd64 -t issue .
```

```
Dockerfile:1
--------------------
   1 | >>> IMAGE httpd:2.4.43
   2 |     
   3 |     ADD sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
--------------------
ERROR: failed to build: failed to solve: dockerfile parse error on line 1: unknown instruction: IMAGE
```

Vamos a modificar **IMAGE** por **FROM**.
```
sudo vi Dockerfile
```

Finalmente el Dockerfile quedo
```
FROM httpd:2.4.43

RUN sed -i 's/Listen 80/Listen 8080/g' /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/
```
