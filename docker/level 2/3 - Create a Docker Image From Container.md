```
One of the Nautilus developer was working to test new changes on a container. He wants to keep a backup of his changes to the container. A new request has been raised for the DevOps team to create a new image from this container. Below are more details about it:

  

a. Create an image `ecommerce:devops` on `Application Server 3` from a container `ubuntu_latest` that is running on same server.
```
# 1. Ingresar al servidor
```bash
ssh banner@stapp03
```
# 2. Listar contenedores
```bash
docker ps
```
# 3. Listar imagenes
```bash
docker images
```
# 4. Crear imagen del contenedor
```bash
docker commit ubuntu_latest ecommerce:devops 
```
# 5. Verificar
```bash
docker images
```
