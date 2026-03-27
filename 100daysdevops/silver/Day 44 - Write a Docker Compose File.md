```
The Nautilus application development team shared static website content that needs to be hosted on the httpd web server using a containerised platform. The team has shared details with the DevOps team, and we need to set up an environment according to those guidelines. Below are the details:



a. On App Server 3 in Stratos DC create a container named httpd using a docker compose file /opt/docker/docker-compose.yml (please use the exact name for file).


b. Use httpd (preferably latest tag) image for container and make sure container is named as httpd; you can use any name for service.


c. Map 80 number port of container with port 6100 of docker host.


d. Map container's /usr/local/apache2/htdocs volume with /opt/finance volume of docker host which is already there. (please do not modify any data within these locations).
```

# Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Create docker compose
```
sudo touch /opt/docker/docker-compose.yml
```

## Modificar docker-compose.yml utiliznado la imagen httpd, nombrando el container httpd y el servicio con cualquier nombre
```
services:
  prueba:
    image: httpd:latest
    container_name: httpd
```

## Modificar docker-compose.yml para el puerto 8085
```
services:
  prueba:
    image: httpd:latest
    container_name: httpd
    ports:
      - "8080:80"
```

## Modificar docker-compose.yml para el volumen
Este es el docker-compose.yml final.

```
sudo vi /opt/docker/docker-compose.yml
```

```
services:
  prueba:
    image: httpd:latest
    container_name: httpd
    ports:
      - "6100:80"
    volumes:
      - /opt/finance:/usr/local/apache2/htdocs
    
```
