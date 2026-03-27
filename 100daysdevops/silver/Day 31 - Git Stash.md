```
The Nautilus application development team was working on a git repository /usr/src/kodekloudrepos/demo present on Storage server in Stratos DC. One of the developers stashed some in-progress changes in this repository, but now they want to restore some of the stashed changes. Find below more details to accomplish this task:



Look for the stashed changes under /usr/src/kodekloudrepos/demo git repository, and restore the stash with stash@{1} identifier. Further, commit and push your changes to the origin.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Ir al directorio
```
cd /usr/src/kodekloudrepos/demo
```

## Ver el estado del repo
```
sudo git status
```

## Listar stash
```
sudo git stash list
```

## Aplicar stash especifico
```
sudo git stash apply stash@{1}
```

## Agregrar los cambios al area de stage
```
sudo git commit -m "Agregando los cambios del stash 1"
```

## Subir cambios al repositorio remoto
```
sudo git push
```
