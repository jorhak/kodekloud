```
The xFusionCorp development team added updates to the project that is maintained under /opt/media.git repo and cloned under /usr/src/kodekloudrepos/media. Recently some changes were made on Git server that is hosted on Storage server in Stratos DC. The DevOps team added some new Git remotes, so we need to update remote on /usr/src/kodekloudrepos/media repository as per details mentioned below:


a. In /usr/src/kodekloudrepos/media repo add a new remote dev_media and point it to /opt/xfusioncorp_media.git repository.


b. There is a file /tmp/index.html on same server; copy this file to the repo and add/commit to master branch.


c. Finally push master branch to this new remote origin.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Ir al directorio /usr/src/kodekloudrepos/media
```
cd /usr/src/kodekloudrepos/media
```

## Agregar nuevo repositorio remoto
```
sudo git remote add dev_media /opt/xfusioncorp_media.git
```

## Copiar fichero
```
sudo cp /tmp/index.html .
```

## Agregarlo al workspace y stage
```
sudo git add .
sudo git commit -m "Agrengando index.html"
```

## Subir al nuevo repositorio remoto
```
sudo git push -u dev_media master
```
