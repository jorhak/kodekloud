```
The Nautilus application development team was working on a git repository /opt/apps.git which is cloned under /usr/src/kodekloudrepos directory present on Storage server in Stratos DC. The team want to setup a hook on this repository, please find below more details:



Merge the feature branch into the master branch`, but before pushing your changes complete below point.

Create a post-update hook in this git repository so that whenever any changes are pushed to the master branch, it creates a release tag with name release-2023-06-15, where 2023-06-15 is supposed to be the current date. For example if today is 20th June, 2023 then the release tag must be release-2023-06-20. Make sure you test the hook at least once and create a release tag for today's release.

Finally remember to push your changes.
Note: Perform this task using the natasha user, and ensure the repository or existing directory permissions are not altered.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```
## Crear hook

```
cd /opt/apps.git
```

```
mkdir -p hooks
```

```
vi hooks/post-receive
```

```
#!/bin/sh

# Lee las referencias actualizadas por el 'push' a través de STDIN
while read oldrev newrev refname; do
    
    # Verifica si la referencia actualizada es 'refs/heads/master'
    if [ "$refname" = "refs/heads/master" ]; then
        
        # Obtener la fecha actual.
        CURRENT_DATE=$(date +%Y-%m-%d)
        TAG_NAME="release-${CURRENT_DATE}"
        
        # 1. Crear el tag anotado, apuntando al nuevo commit ($newrev).
        # Ya que estamos en el repositorio 'bare', esto guarda el tag permanentemente.
        git tag -a "$TAG_NAME" -m "Release tag for $CURRENT_DATE" "$newrev" 
        
        # 2. Imprimir mensaje de éxito al cliente que hizo el push.
        echo "================================================================"
        echo "AUTOMATIC RELEASE TAG CREATED:"
        echo "Tag: $TAG_NAME, apuntando a commit: $newrev"
        echo "================================================================"
        
        # Si ya existe un tag con ese nombre (por haber pusheado el mismo día),
        # 'git tag' fallará, pero no queremos que el hook falle completamente.
        # Por simplicidad, ignoramos el código de salida de 'git tag'.
    fi
done

exit 0
```

## Dar permisos de ejecucion
```
chmod +x hooks/post-receive
```

## Configurar email y user
```
git config --global user.email "nat@kodekloud.com"
git config --global user.name "Natasha Romanof"
```
## Vamos a directorio
```
cd /usr/src/kodekloudrepos/apps
```
## Nos ubicamos sobre la rama master
```
git switch master
```

## Merge sobre feature
```
git merge feature
```

## Subir cambios al repositorio
```
git push
```
## Verificar que el hook se ejecuto
```
git fetch --tags
git tag -l
```
