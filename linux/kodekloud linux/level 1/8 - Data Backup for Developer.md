```
Within the Stratos DC, the Nautilus storage server hosts a directory named /data, serving as a repository for various developers non-confidential data. Developer mark has requested a copy of their data stored in /data/mark. The System Admin team has provided the following steps to fulfill this request:



a. Create a compressed archive named mark.tar.gz of the /data/mark directory.

b. Transfer the archive to the /home directory on the Storage Server.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Backup el directorio
```
tar czvf mark.tar.gz /data/mark
```

## Mover al directoiro /home
```
mv mark.tar.gz /home/
```

## Restaurar Backup
```
tar xzvf mark.tar.gz
```