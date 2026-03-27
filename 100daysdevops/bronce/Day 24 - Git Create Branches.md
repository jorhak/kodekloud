```
Nautilus developers are actively working on one of the project repositories, /usr/src/kodekloudrepos/blog. Recently, they decided to implement some new features in the application, and they want to maintain those new changes in a separate branch. Below are the requirements that have been shared with the DevOps team:



1.-On Storage server in Stratos DC create a new branch xfusioncorp_blog from master branch in /usr/src/kodekloudrepos/blog git repo.


2.-Please do not try to make any changes in the code.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

### Ir al repositorio
```
cd /usr/src/kodekloudrepos/blog
```

### Ver en que rama se encuentra
```
sudo git branch
```
Debe estar en la rama **master**.

### Cambiar de rama 
```
sudo git checkout master
```
### Crear rama
```
sudo git checkout -b xfusioncorp_blog
```
