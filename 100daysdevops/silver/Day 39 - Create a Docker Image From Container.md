```
One of the Nautilus developer was working to test new changes on a container. He wants to keep a backup of his changes to the container. A new request has been raised for the DevOps team to create a new image from this container. Below are more details about it:

  

a. Create an image `demo:datacenter` on `Application Server 1` from a container `ubuntu_latest` that is running on same server.
```

## Ingresar al servidor
```
ssh tony@172.16.238.10
```

## Listar las containers
```
docker ps | grep ubuntu_latest
```

## Listar imagenes
```
docker images
```
## Crear imagen desde un contenedor
```
docker commit <ID_COMMIT> demo:datacenter
docker commit b1388fb3d434 demo:datacenter
```

De esta forma hemos creado una imagen a partir de un contenedor.