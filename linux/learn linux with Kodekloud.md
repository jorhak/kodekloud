# 2 Group Creation and User Assignment
## Crear grupo
```
sudo groupadd nautilus_noc
```

## Crear usuario y asignar grupo
```
sudo useradd -g nautilus_noc -c "Hola Sonya" -d /home/sonya -s /bin/bash sonya
```

# 3 Linux User Setup with Non-Interactive Shell
```
sudo useradd -s /sbin/nologin john
```

# 4 Service User Creation without Home Directory
```
sudo useradd -M mark
```

# 5 Temporary User Setup with Expiry
```
sudo useradd -e 2024-01-28 -c "Mari Yam" -d /home/mariyam -s /bin/bash mariyam
```

# 6 Linux User Data Transfer
```
Due to an accidental data mix-up, user data was unintentionally mingled on Nautilus App Server 3 at the /home/usersdata location by the Nautilus production support team in Stratos DC. To rectify this, specific user data needs to be filtered and relocated. Here are the details:



Locate all files (excluding directories) owned by user rose within the /home/usersdata directory on App Server 3. Copy these files while preserving the directory structure to the /media directory.
```

## Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Buscar todos los ficheros del propietario mark y copiarlos en la ruta /media
```
sudo find /home/usersdata -type f -user rose -exec cp --parents {} /media/ \;
```
Este comando busca a todos los ficheros que son propiedad del usuario **rose**, lo hace de forma recursiva.
Ahora si solo queremos que solo busque a nivel del padre y no de forma recursiva debemos ejecutar:
```
sudo find /home/usersdata -maxdepth 1 -type f -user mark -exec cp --parents {} /media/ \;
```
