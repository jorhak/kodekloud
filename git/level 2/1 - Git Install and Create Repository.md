```
The Nautilus development team shared with the DevOps team requirements for new application development, setting up a Git repository for that project. Create a Git repository on `Storage server` in Stratos DC as per details given below:  
  

  

1. Install `git` package using `yum` on `Storage server`.  
      
    
2. After that, create/init a git repository named `/opt/beta.git` (use the exact name as asked and make sure not to create a bare repository).
```
# Ingresar al servidor
```bash
ssh natasha@ststor01
```
# Instalar git
```bash
sudo yum update -y && sudo yum install -y git
```
#### Verificar si se instalo
```bash
git --version
```
# Crear repositorio
```bash
sudo mkdir /opt/ecommerce.git
cd /opt/ecommerce.git
```
#### Inicializar repositorio
```bash
sudo git init
```
# Verificar
```bash
ls -la
```
