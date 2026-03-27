```
The Nautilus application development team has been working on a project repository `/opt/ecommerce.git`. This repo is cloned at `/usr/src/kodekloudrepos` on `storage server` in `Stratos DC`. They recently shared the following requirements with DevOps team:  
  

  

Create a new branch `datacenter` in `/usr/src/kodekloudrepos/ecommerce` repo from `master` and copy the `/tmp/index.html` file (present on `storage server` itself) into the repo. Further, `add/commit` this file in the new branch and merge back that branch into `master` branch. Finally, push the changes to the origin for both of the branches.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Nos dirigimos al directorio donde este el repositorio
```
cd /usr/src/kodekloudrepos/ecommerce
```

## Verificar rama
```
sudo git branch
```
Debe ser la master.

## Cambiar de rama (sino esta en la rama master)
```
sudo git checkout master
```

## Crear rama
```
sudo git checkout -b datacenter
```

## Copiar fichero
```
sudo cp /tmp/index.html .
```

## Agregar los cambios al workspace y stage
```
sudo git add .
sudo git commit -m "Add index.html"
```

## Subir cambios de la rama datacenter
```
sudo git branch -M datacenter
sudo git push -u origin datacenter
```

## Realizar merge de datacenter a master
```
sudo git checkout master
sudo git merge datacenter
sudo git push
```