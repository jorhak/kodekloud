```
The Nautilus application development team has been working on a project repository `/opt/official.git`. This repo is cloned at `/usr/src/kodekloudrepos` on `storage server` in `Stratos DC`. They recently shared the following requirements with DevOps team:  
  

  

Create a new branch `devops` in `/usr/src/kodekloudrepos/official` repo from `master` and copy the `/tmp/index.html` file (present on `storage server` itself) into the repo. Further, `add/commit` this file in the new branch and merge back that branch into `master` branch. Finally, push the changes to the origin for both of the branches.
```
# 1. Ingresar al servidor
```bash
ssh natasha@ststor01
```
# 2. Ir al repositorio
```bash
cd /opt/news.git
ls
```
## 2.1 Ir donde fue clonado
```bash
cd /usr/src/kodekloudrepos/news
ls
```
## 2.2 Ver el estado en el que se encuentra
```bash
git status
```
No indica que debemos ejecutar un comando:
```bash
git config --global --add safe.directory /usr/src/kodekloudrepos/news
```
## 2.3 Ver cuantas ramas tiene
```bash
git branch
```
Solo tiene una rama y nos encontramos en la rama master.
# 3. Crear rama datacenter
```bash
sudo git checkout -b datacenter
git branch
```
## 3.1 Copiar /tmp/index.html al repositorio
```bash
sudo cp /tmp/index.html .
ls
git status
```
## 3.2 Comenzar a realziar el seguimiento del fichero
```bash
sudo git add index.html
```
## 3.3 Realizar commit
```bash
sudo git commit -m "Se agrego el fichero index.html."
```
## 3.4 Subir a repositorio
```bash
sudo git push
```
Nos indica que debemos ejecutar el comando:
```bash
sudo git push --set-upstream origin datacenter
```
## 3.5 Ver que la rama se subio
```bash
git branch -r
```
# 4. Realizar merge de datacenter a master
## 4.1 Cambiar a la rama master
Para realizar el merge debemos cambiarnos de rama a **master**, esta es la que va fusionar con **datacenter** y de esta forma **master** va tener los ficheros de **datacenter**.
```bash
sudo git checkout master
git branch
ls
```
## 4.2 Merge
```bash
sudo git merge datacenter
ls
```
## 4.3 Ver estado
```bash
git status
```
## 4.4 Subir al repositorio
```bash
sudo git push
```
# 5. Verificar
```bash
ls -la
git log -n3
git branch
```