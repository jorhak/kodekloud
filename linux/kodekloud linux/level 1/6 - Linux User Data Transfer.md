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