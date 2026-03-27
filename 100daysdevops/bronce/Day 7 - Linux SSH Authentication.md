Primero debemos ubicarnos en el servidor del cual nos vamos a conectar a los otros servidores en nuestro caso es el servidor **jump_host**

```
ssh thor@jump_host
```

## Configurar autenticacion sin contrasena
```
sudo visudo
```

### Agregar una linea similar reemplazando sudo_user por el usuario thor
```
thor ALL=(ALL) NOPASSWD: ALL
```

## Generar llaves
```
ssh-keygen -t rsa -b 4096
```

### Copiar la llave publica en todos los servidores
```
ssh-copy-id -i ~/.ssh/id_rsa.pub tony@172.16.238.10
ssh-copy-id -i ~/.ssh/id_rsa.pub steve@172.16.238.11
ssh-copy-id -i ~/.ssh/id_rsa.pub banner@172.16.238.12
```

## Probar conexion
```
ssh tony@172.16.238.10
ssh steve@172.16.238.11
ssh banner@172.16.238.12
```
