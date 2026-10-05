```
As per recent requirements shared by the Nautilus application development team, they need custom images created for one of their projects. Several of the initial testing requirements are already been shared with DevOps team. Therefore, create a docker file /opt/docker/Dockerfile (please keep D capital of Dockerfile) on App server 3 in Stratos DC and configure to build an image with the following requirements:

  
  
  

a. Use ubuntu:24.04 as the base image.

  
  

b. Install apache2 and configure it to work on 3002 port. (do not update any other Apache configuration settings like document root etc).
```
# Ingresar al servidor
```bash
ssh steve@stapp02
```
# Ir al directorio
```bash
cd /opt/docker
```
# Crear DockerFile
```bash
sudo vi Dockerfile
```

```
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && \
    apt install -y apache2 && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

RUN sed -i 's/Listen 80/Listen 6300/' /etc/apache2/ports.conf

EXPOSE 6300
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
# Construir imagen
```bash
docker buildx build -t apache2:1.0.0 .
```
# Verificacion
```bash
docker run -d -p80:6300 --name apa apache2:1.0.0
curl localhost
```