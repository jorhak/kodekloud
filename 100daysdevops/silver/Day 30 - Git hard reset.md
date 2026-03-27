```
The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/media` present on `Storage server` in `Stratos DC`. This was just a test repository and one of the developers just pushed a couple of changes for testing, but now they want to clean this repository along with the commit history/work tree, so they want to point back the `HEAD` and the branch itself to a commit with message `add data.txt file`. Find below more details:  
  

  

1. In `/usr/src/kodekloudrepos/media` git repository, reset the git commit history so that there are only two commits in the commit history i.e `initial commit` and `add data.txt file`.  
      
    
2. Also make sure to push your changes.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Ir al directorio
```
cd /usr/src/kodekloudrepos/media
```

## Ver el estado del repositorio
```
sudo git status
```

## Ver los logs del repositorio
```
sudo git log --oneline -n 15
```

## Deshacer los commits y dejar solo los dos primeros
```
sudo git reset --hard HEAD~10
```

## Forzar los cambios en el repositorio remoto

```
sudo git push --force
```
