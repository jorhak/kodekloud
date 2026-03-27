```
Nautilus project developers are planning to start testing on a new project. As per their meeting with the DevOps team, they want to test containerized environment application features. As per details shared with DevOps team, we need to accomplish the following task:


a. Pull busybox:musl image on App Server 2 in Stratos DC and re-tag (create new tag) this image as busybox:local.
```
# Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Descargar imagen
```
docker pull busybox:musl
```

## Opcion 1
```
docker tag busybox:musl busybox:local
```
## Opcion 2 Crear Dockerfile
```
vi Dockerfile
```

```
FROM busybox:musl
```

### Crear imagen con nuevo tag
```
docker buildx build --platform linux/amd64 -t busybox:local .
```

## Verificar que se creo la imagen
```
docker images | grep busybox
```
