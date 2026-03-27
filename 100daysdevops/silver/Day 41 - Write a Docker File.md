```
As per recent requirements shared by the Nautilus application development team, they need custom images created for one of their projects. Several of the initial testing requirements are already been shared with DevOps team. Therefore, create a docker file /opt/docker/Dockerfile (please keep D capital of Dockerfile) on App server 3 in Stratos DC and configure to build an image with the following requirements:



a. Use ubuntu:24.04 as the base image.


b. Install apache2 and configure it to work on 8088 port. (do not update any other Apache configuration settings like document root etc).
```

# Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Editar Dockerfile
```
mkdir /opt/docker
sudo vi /opt/docker/Dockerfile
sudo chmod o+x /opt/docker/Dockerfile
```


```
FROM ubuntu:24.04
RUN apt update && \
    DEBIAN_FRONTEND=noninteractive && \
    apt install -y apache2 && \
    rm -rf /var/lib/apt/lists/*
RUN sed -i 's/Listen 80/Listen 8088/' /etc/apache2/ports.conf
EXPOSE 8088
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Me costo crear esta imagen ya que tenia que utilizar explicitamente **DEBIAN_FRONTEND=noninteractive**, porque de lo contrario no podia hacer la instalacion de apache2.
Previo a esto me cre un contenedor:
```
docker run -it --name hola ubuntu:24.04
```

Y ejecute los comandos que normalmente ejecuto para realizar la instalacion de **apache2**, y todo salia bien, cuando intentaba replicar el orden de la ejecucion de los comandos me salia error. Sin embargo cuando agrege **DEBIAN_FRONTEND** esto error se soluciono.
## Construir imagen 
```
cd /opt/docker
docker buildx build --platform linux/amd64 -t dia41 .
```
