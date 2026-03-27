```
Sarah and Max were working on writting some stories which they have pushed to the repository. Max has recently added some new changes and is trying to push them to the repository but he is facing some issues. Below you can find more details:


SSH into storage server using user max and password Max_pass123. Under /home/max you will find the story-blog repository. Try to push the changes to the origin repo and fix the issues. The story-index.txt must have titles for all 4 stories. Additionally, there is a typo in The Lion and the Mooose line where Mooose should be Mouse.


Click on the Gitea UI button on the top bar. You should be able to access the Gitea page. You can login to Gitea server from UI using username sarah and password Sarah_pass123 or username max and password Max_pass123.


Note: For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

## Ingresar al servidor
```
ssh max@172.16.238.15
```

## Ingresar al directorio
```
cd story-blog
```

## Ver el estado del repo y ver los ficheros
```
sudo git status
```

## Subir los cambios al repositorio
```
sudo git push
```

```
To http://git.stratos.xfusioncorp.com/sarah/story-blog.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'http://git.stratos.xfusioncorp.com/sarah/story-blog.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
## Traer los cambios del repositorio remoto

```
sudo git pull
```
Aqui vamos a tener el error:
```
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From http://git.stratos.xfusioncorp.com/sarah/story-blog
   9edf77a..3b8c996  master     -> origin/master
Auto-merging story-index.txt
CONFLICT (add/add): Merge conflict in story-index.txt
Automatic merge failed; fix conflicts and then commit the result.
```

## Editar el fichero story-index.txt
#### Ver su contenido
```
cat story-index.txt 
```

```
<<<<<<< HEAD
1. The Lion and the Mooose
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
=======
5. The Lion and the Mouse
6. The Frogs and the Ox
7. The Fox and the Grapes
>>>>>>> 3b8c996e00bbfda20ba45ca69086d59380826e58
```

#### Editar fichero
```
sudo vi story-index.txt
```

```
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
```

## Agregar los cambios a las areas workspace y stage
```
sudo git add .
sudo git commit -m "Solucionando conflictos"
```

## Agregar usuario
```
sudo git config --global user.name "Max"
sudo git config --global user.email max@kodekloud.com
```

## Subir cambios al repositorio remoto
```
sudo git push
```

## Ver los logs de los commits
```
sudo git log --oneline -n 10
```
