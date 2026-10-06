```
Nautilus developers are actively working on one of the project repositories, `/usr/src/kodekloudrepos/games`. Recently, they decided to implement some new features in the application, and they want to maintain those new changes in a separate branch. Below are the requirements that have been shared with the DevOps team:  
  

  

1. On `Storage server` in Stratos DC create a new branch `xfusioncorp_games` from `master` branch in `/usr/src/kodekloudrepos/games` git repo.  
      
    
2. Please do not try to make any changes in the code.
```
# 1. Ingresar al servidor
```bash
ssh natasha@ststor01
```
# 2. Ir al directorio
```bash
cd /usr/src/kodekloudrepos/games
git status
```
Nos pide que ejecutemos este comando para tomarlo como directorio seguro:
```bash
git config --global --add safe.directory /usr/src/kodekloudrepos/games
git status
git branch
```
# 3. Crear nueva rama a partir de master
```bash
sudo git checkout master
sudo git checkout -b xfusioncorp_games
```
# 4. Verificar
```bash
git branch
```