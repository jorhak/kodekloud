```
The DevOps team established a new Git repository last week, which remains unused at present. However, the Nautilus application development team now requires a copy of this repository on the `Storage Server` in the Stratos DC. Follow the provided details to clone the repository:  
  

  

1. The repository to be cloned is located at `/opt/official.git`  
      
    
2. Clone this Git repository to the `/usr/src/kodekloudrepos` directory. Perform this task using the natasha user, and ensure that no modifications are made to the repository or existing directories, such as changing permissions or making unauthorized alterations.
```

1. Ingresar al servidor
```
ssh natasha@ststor01
```

2. Ir al directorio
```
cd /usr/src/kodekloudrepos
```

3. Clonar repositorio
```
git clone /opt/official.git
```
3. Verificar
```
ls
```
