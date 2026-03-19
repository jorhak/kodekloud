# 1 Set Up Git Repository on Storage Server
```
The Nautilus development team has provided requirements to the DevOps team for a new application development project, specifically requesting the establishment of a Git repository. Follow the instructions below to create the Git repository on the `Storage server` in the Stratos DC:  
  

  

1. Utilize `yum` to install the `git` package on the `Storage Server`.  
      
    
2. Create a bare repository named `/opt/beta.git` (ensure exact name usage).
```

1. Instalar Git
- Ingresar al servidor
```
ssh natasha@ststor01
```

- Intalar
```
sudo yum update -y && sudo yum install git -y
```

- Verificar instalacion
```
git -v
```

2. Crear repositorio
```
cd /opt
sudo git init --bare demo.git
```

 git bare repository '/opt/apps.git' not found on Storage Server or its not a bare repository

# 2 Clone Git Repository on Storage Server
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

- '/opt/official.git' git repository is not cloned under '/usr/src/kodekloudrepos/' on Storage Server


- '/opt/apps.git' git repository is not cloned under '/usr/src/kodekloudrepos/' on Storage Server