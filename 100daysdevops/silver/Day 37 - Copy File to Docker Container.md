```
The Nautilus DevOps team possesses confidential data on App Server 2 in the Stratos Datacenter. A container named ubuntu_latest is running on the same server.



Copy an encrypted file /tmp/nautilus.txt.gpg from the docker host to the ubuntu_latest container located at /opt/. Ensure the file is not modified during this operation.
```
## Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Copiar del host al container
```
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/
```

## Verificar que se copio
```
docker exec ubuntu_latest ls /opt
```
