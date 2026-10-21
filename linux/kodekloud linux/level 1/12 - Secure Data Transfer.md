```
A Nautilus developer has stored confidential data on the jump host within Stratos DC. To ensure security and compliance, this data must be transferred to one of the app servers. Given developers lack direct access to these servers, the system admin team has been enlisted for assistance.



Copy /tmp/nautilus.txt.gpg file from jump server to App Server 1 placing it in the directory /home/nfsshare.
```

## Abrir otra terminal
En esta terminal vamos a ingresar a **App Server 1**
```
ssh tony@172.16.238.10
```
## Copiar fichero a otro servidor
Aqui estamos en el servidor **jump server**
```
scp /tmp/nautilus.txt.gpg tony@172.16.238.10:/home/nfsshare
```
## Verificar que se copio en App Server 1
Nos vamos a la terminal que abrimos anteriormente
```
ls -la /home/nfsshare
```