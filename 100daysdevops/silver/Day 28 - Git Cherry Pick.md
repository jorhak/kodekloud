```
The Nautilus application development team has been working on a project repository /opt/games.git. This repo is cloned at /usr/src/kodekloudrepos on storage server in Stratos DC. They recently shared the following requirements with the DevOps team:



There are two branches in this repository, master and feature. One of the developers is working on the feature branch and their work is still in progress, however they want to merge one of the commits from the feature branch to the master branch, the message for the commit that needs to be merged into master is Update info.txt. Accomplish this task for them, also remember to push your changes eventually.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Ir al directorio
```
cd /usr/src/kodekloudrepos/games
ls
```

## Ver estado del repositorio
```
sudo git status
```

## Ver la ramas
```
sudo git branch
```

Debemos estar en la rama **feature** de lo contrario debemos cambiarnos
```
sudo git checkout feature
```
## Ver commits disponibles
```
sudo git log --oneline --graph -n 5
```

Necesitamos el commit **Update info.txt**, debemos guardar su hash.

## Cambiar a la rama master
```
sudo git switch master
```

## Aplicar commit especifico
```
sudo git cherry-pick 9b32b32
```
## Subir cambios
```
sudo git push
```
