```
Nautilus project developers are planning to start testing on a new project. As per their meeting with the DevOps team, they want to test containerized environment application features. As per details shared with DevOps team, we need to accomplish the following task:

  

a. Pull `busybox:musl` image on `App Server 1` in Stratos DC and re-tag (create new tag) this image as `busybox:local`.
```
# 1 Ingresar a servidor
```bash
ssh tony@stapp01
```
# 2 Descargar imagen
```bash
docker pull busybox:musl
```
# 3 Crear nuevo tag
```bash
docker tag busybox:musl busybox:local
```
# Verificar
```bash
docker images
```