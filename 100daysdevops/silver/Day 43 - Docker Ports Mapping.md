```
The Nautilus DevOps team is planning to host an application on a nginx-based container. There are number of tickets already been created for similar tasks. One of the tickets has been assigned to set up a nginx container on Application Server 3 in Stratos Datacenter. Please perform the task as per details mentioned below:


a. Pull nginx:alpine docker image on Application Server 3.


b. Create a container named media using the image you pulled.


c. Map host port 6100 to container port 80. Please keep the container in running state.
```

## Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Descargar imagen
```
docker pull nginx:alpine
```

## Crear con el container con nombre **media** y puerto **6100**.
```
docker run -d -p 6100:80 --name media nginx:alpine
```
