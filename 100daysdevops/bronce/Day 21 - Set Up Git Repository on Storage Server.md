```
The Nautilus development team has provided requirements to the DevOps team for a new application development project, specifically requesting the establishment of a Git repository. Follow the instructions below to create the Git repository on the Storage server in the Stratos DC:



Utilize yum to install the git package on the Storage Server.


Create a bare repository named /opt/official.git (ensure exact name usage).
```
## Acceder al servidor
```
ssh natasha@172.16.238.15
```

## Instalar git
```
sudo dnf install git -y
```

## Crear repositorio
```
sudo git init --bare /opt/official.git
```
