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