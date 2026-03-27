```
The DevOps team established a new Git repository last week, which remains unused at present. However, the Nautilus application development team now requires a copy of this repository on the Storage Server in the Stratos DC. Follow the provided details to clone the repository:



The repository to be cloned is located at /opt/ecommerce.git


Clone this Git repository to the /usr/src/kodekloudrepos directory. Perform this task using the natasha user, and ensure that no modifications are made to the repository or existing directories, such as changing permissions or making unauthorized alterations.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Clonar repositorio
### Ir al directorio
```
cd /usr/src/kodekloudrepos
```

## Clonar repo

```
git clone /opt/ecommerce.git
```
