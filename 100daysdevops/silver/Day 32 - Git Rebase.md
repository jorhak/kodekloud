```
The Nautilus application development team has been working on a project repository `/opt/cluster.git`. This repo is cloned at `/usr/src/kodekloudrepos` on `storage server` in `Stratos DC`. They recently shared the following requirements with DevOps team:  
  

  

One of the developers is working on `feature` branch and their work is still in progress, however there are some changes which have been pushed into the `master` branch, the developer now wants to `rebase` the `feature` branch with the `master` branch without loosing any data from the `feature` branch, also they don't want to add any `merge commit` by simply merging the `master` branch into the `feature` branch. Accomplish this task as per requirements mentioned.  
  

Also remember to push your changes once done.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Ir al directorio
```
cd /usr/src/kodekloudrepos/cluster
```

## Ver el estado del repositorio
```
sudo git status
```

## Ver los logs de los commits
```
sudo git log --oneline -n 15
```

## Ver en que rama estamos
```
sudo git branch
```
Debemos estar en la rama **feature**, cambiar de rama si es que nos encontramos en otra rama.

```
sudo git switch feature
```

## Rebasar sobre master
Para hacer el **rebase** lo que debemos hacer es ubicarnos sobre la rama donde no se tienen los cambios que se han hecho en otra rama.
Lo que hace es que nosotros estamos en nuestra rama trabajando y de igual modo los demas siguen trabajando sobre sus ramas y ahora necesito que sus cambios esten en mi rama para eso se utiliza el **rebase**.
```
sudo git rebase master
```

## Subir cambios al repositorio remoto
```
sudo git push --force-with-lease --set-upstream origin feature
```
