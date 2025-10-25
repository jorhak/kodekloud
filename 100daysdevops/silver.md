Entramos al reto de 100 dias de DevOps, aqui se van a tener los retos de los dias 26 hasta el dia 50.

# Day 26: Git Manage Remotes
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

# Day 27: Git Revert Some Changes
```
The Nautilus application development team was working on a git repository /usr/src/kodekloudrepos/news present on Storage server in Stratos DC. However, they reported an issue with the recent commits being pushed to this repo. They have asked the DevOps team to revert repo HEAD to last commit. Below are more details about the task:


In /usr/src/kodekloudrepos/news git repository, revert the latest commit ( HEAD ) to the previous commit (JFYI the previous commit hash should be with initial commit message ).


Use revert news message (please use all small letters for commit message) for the new revert commit.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Ir al directorio
```
cd /usr/src/kodekloudrepos/news
```

## Ver el estado del repositorio
```
sudo git status
```
## Ver logs del repositorio
```
sudo git log --oneline -n 5
```
## Revertir al primer commit
```
sudo git revert HEAD
```

# Day 28: Git Cherry Pick
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

# Dia 29
```
`Max` want to push some new changes to one of the repositories but we don't want people to push directly to `master` branch, since that would be the final version of the code. It should always only have content that has been reviewed and approved. We cannot just allow everyone to directly push to the master branch. So, let's do it the right way as discussed below:

  

SSH into `storage server` using user `max`, password `Max_pass123` . There you can find an already cloned repo under `Max` user's home.  
  

Max has written his story about The 🦊 Fox and Grapes 🍇  
  

Max has already pushed his story to remote git repository hosted on `Gitea` branch `story/fox-and-grapes`  
  

Check the contents of the cloned repository. Confirm that you can see Sarah's story and history of commits by running `git log` and validate author info, commit message etc.  
  

Max has pushed his story, but his story is still not in the `master` branch. Let's create a Pull Request(PR) to merge Max's `story/fox-and-grapes` branch into the `master` branch  
  

Click on the `Gitea UI` button on the top bar. You should be able to access the `Gitea` page.  
  

UI login info:

- Username: `max`

- Password: `Max_pass123`

PR title : `Added fox-and-grapes story`

PR pull from branch: `story/fox-and-grapes` (source)

PR merge into branch: `master` (destination)  
  

Before we can add our story to the `master` branch, it has to be reviewed. So, let's ask `tom` to review our PR by assigning him as a reviewer  
  
  
Add tom as reviewer through the Git Portal UI

- Go to the newly created PR
    
- Click on Reviewers on the right
    
- Add tom as a reviewer to the PR
    

Now let's review and approve the PR as user `Tom`  
  
  
Login to the portal with the user `tom`  
  

Logout of `Git Portal UI` if logged in as `max`  
  

UI login info:

- Username: `tom`

- Password: `Tom_pass123`

PR title : `Added fox-and-grapes story`

Review and merge it.

Great stuff!! The story has been merged! 👏  
  

`Note:` For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

## Ingresar al servidor
```
ssh max@172.16.238.15
```

```
cd story-blog/
```

```
git remote -v
```

```
git log
```

# Day 30: Git hard reset
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

# Day 31: Git Stash
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

# Day 32: Git Rebase
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

# Day 33: Resolve Git Merge Conflicts
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

# Day 34: Git Hook
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

# Day 35: Install Docker Packages and Start Docker Service
```
The Nautilus DevOps team aims to containerize various applications following a recent meeting with the application development team. They intend to conduct testing with the following steps:


Install docker-ce and docker compose packages on App Server 1.


Initiate the docker service.
```

## Ingresar al servidor
```
ssh tony@172.16.238.10
```

## Instalar docker y docker compose
### Agregar repositorio
```
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### Instalar Docker Engine
```
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Iniciar Docker Engine
```
sudo systemctl enable --now docker
```

### Verificar estado
```
sudo systemctl status docker
```

### Version de Docker y Docker Compose
```
docker --version
docker compose version
```

# Day 36: Deploy Nginx Container on Application Server
```
The Nautilus DevOps team is conducting application deployment tests on selected application servers. They require a nginx container deployment on Application Server 2. Complete the task with the following instructions:


On Application Server 2 create a container named nginx_2 using the nginx image with the alpine tag. Ensure container is in a running state.
```

## Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Crear container
```
docker run -d -p 80:80 --name nginx_2 nginx:alpine
```

### Ver estado del container
```
docker ps 
```

# Day 37: Copy File to Docker Container
```
The Nautilus DevOps team possesses confidential data on App Server 2 in the Stratos Datacenter. A container named ubuntu_latest is running on the same server.



Copy an encrypted file /tmp/nautilus.txt.gpg from the docker host to the ubuntu_latest container located at /opt/. Ensure the file is not modified during this operation.
```
## Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Copiar del host al container
```
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/opt/
```

## Verificar que se copio
```
docker exec ubuntu_latest ls /opt
```

# Day 38: Pull Docker Image
```
Nautilus project developers are planning to start testing on a new project. As per their meeting with the DevOps team, they want to test containerized environment application features. As per details shared with DevOps team, we need to accomplish the following task:


a. Pull busybox:musl image on App Server 2 in Stratos DC and re-tag (create new tag) this image as busybox:local.
```
# Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Descargar imagen
```
docker pull busybox:musl
```

## Opcion 1
```
docker tag busybox:musl busybox:local
```
## Opcion 2 Crear Dockerfile
```
vi Dockerfile
```

```
FROM busybox:musl
```

### Crear imagen con nuevo tag
```
docker buildx build --platform linux/amd64 -t busybox:local .
```

## Verificar que se creo la imagen
```
docker images | grep busybox
```

# Day 39: Create a Docker Image From Container
```
One of the Nautilus developer was working to test new changes on a container. He wants to keep a backup of his changes to the container. A new request has been raised for the DevOps team to create a new image from this container. Below are more details about it:

  

a. Create an image `demo:datacenter` on `Application Server 1` from a container `ubuntu_latest` that is running on same server.
```

## Ingresar al servidor
```
ssh tony@172.16.238.10
```

## Listar las containers
```
docker ps | grep ubuntu_latest
```

## Listar imagenes
```
docker images
```
## Crear imagen desde un contenedor
```
docker commit <ID_COMMIT> demo:datacenter
docker commit b1388fb3d434 demo:datacenter
```

De esta forma hemos creado una imagen a partir de un contenedor.

# Day 40: Docker EXEC Operations
```
One of the Nautilus DevOps team members was working to configure services on a kkloud container that is running on App Server 2 in Stratos Datacenter. Due to some personal work he is on PTO for the rest of the week, but we need to finish his pending work ASAP. Please complete the remaining work as per details given below:


a. Install apache2 in kkloud container using apt that is running on App Server 2 in Stratos Datacenter.


b. Configure Apache to listen on port 6000 instead of default http port. Do not bind it to listen on specific IP or hostname only, i.e it should listen on localhost, 127.0.0.1, container ip, etc.


c. Make sure Apache service is up and running inside the container. Keep the container in running state at the end.
```

## Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Listar containers
```
docker ps
```

## Ingresar al container
```
docker exec -it kkloud bash
```

### Instalar Apache2
```
apt install -y apache2 nano
```

### Configurar puerto
```
nano /etc/apache2/ports.conf
```

```
LISTEN 6000
```

### Reiniciar servicio
```
service apache2 restart
service apache2 status
```

### Verificar que apache este funcionando
```
curl localhost:6000
curl 127.0.0.1:6000
curl <IP>:6000
```

Para conocer la IP tenemos que salir del container y ejecutar:
```
docker inspect kkloud
```

Ahora podemos reemplar la IP.

# Day 41: Write a Docker File
```
As per recent requirements shared by the Nautilus application development team, they need custom images created for one of their projects. Several of the initial testing requirements are already been shared with DevOps team. Therefore, create a docker file /opt/docker/Dockerfile (please keep D capital of Dockerfile) on App server 3 in Stratos DC and configure to build an image with the following requirements:



a. Use ubuntu:24.04 as the base image.


b. Install apache2 and configure it to work on 8088 port. (do not update any other Apache configuration settings like document root etc).
```

# Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Editar Dockerfile
```
mkdir /opt/docker
sudo vi /opt/docker/Dockerfile
sudo chmod o+x /opt/docker/Dockerfile
```


```
FROM ubuntu:24.04
RUN apt update && \
    DEBIAN_FRONTEND=noninteractive && \
    apt install -y apache2 && \
    rm -rf /var/lib/apt/lists/*
RUN sed -i 's/Listen 80/Listen 8088/' /etc/apache2/ports.conf
EXPOSE 8088
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Me costo crear esta imagen ya que tenia que utilizar explicitamente **DEBIAN_FRONTEND=noninteractive**, porque de lo contrario no podia hacer la instalacion de apache2.
Previo a esto me cre un contenedor:
```
docker run -it --name hola ubuntu:24.04
```

Y ejecute los comandos que normalmente ejecuto para realizar la instalacion de **apache2**, y todo salia bien, cuando intentaba replicar el orden de la ejecucion de los comandos me salia error. Sin embargo cuando agrege **DEBIAN_FRONTEND** esto error se soluciono.
## Construir imagen 
```
cd /opt/docker
docker buildx build --platform linux/amd64 -t dia41 .
```


# Day 42: Create a Docker Network
```
The Nautilus DevOps team needs to set up several docker environments for different applications. One of the team members has been assigned a ticket where he has been asked to create some docker networks to be used later. Complete the task based on the following ticket description:

  

a. Create a docker network named as `ecommerce` on App Server `3` in `Stratos DC`.  
  

b. Configure it to use `bridge` drivers.  
  

c. Set it to use subnet `10.10.1.0/24` and iprange `10.10.1.0/24`.
```

## Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Crear red
```
docker network create \
  --driver=bridge \
  --subnet=10.10.1.0/24 \
  --ip-range=10.10.1.0/24 \
  ecommerce
```

# Day 43: Docker Ports Mapping
```
The Nautilus DevOps team is planning to host an application on a nginx-based container. There are number of tickets already been created for similar tasks. One of the tickets has been assigned to set up a nginx container on Application Server 3 in Stratos Datacenter. Please perform the task as per details mentioned below:


a. Pull nginx:alpine docker image on Application Server 3.


b. Create a container named media using the image you pulled.


c. Map host port 6100 to container port 80. Please keep the container in running state.
```

## Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Descargar imagen
```
docker pull nginx:alpine
```

## Crear con el container con nombre **media** y puerto **6100**.
```
docker run -d -p 6100:80 --name media nginx:alpine
```

# Day 44: Write a Docker Compose File
```
The Nautilus application development team shared static website content that needs to be hosted on the httpd web server using a containerised platform. The team has shared details with the DevOps team, and we need to set up an environment according to those guidelines. Below are the details:



a. On App Server 3 in Stratos DC create a container named httpd using a docker compose file /opt/docker/docker-compose.yml (please use the exact name for file).


b. Use httpd (preferably latest tag) image for container and make sure container is named as httpd; you can use any name for service.


c. Map 80 number port of container with port 6100 of docker host.


d. Map container's /usr/local/apache2/htdocs volume with /opt/finance volume of docker host which is already there. (please do not modify any data within these locations).
```

# Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Create docker compose
```
sudo touch /opt/docker/docker-compose.yml
```

## Modificar docker-compose.yml utiliznado la imagen httpd, nombrando el container httpd y el servicio con cualquier nombre
```
services:
  prueba:
    image: httpd:latest
    container_name: httpd
```

## Modificar docker-compose.yml para el puerto 8085
```
services:
  prueba:
    image: httpd:latest
    container_name: httpd
    ports:
      - "8080:80"
```

## Modificar docker-compose.yml para el volumen
Este es el docker-compose.yml final.

```
sudo vi /opt/docker/docker-compose.yml
```

```
services:
  prueba:
    image: httpd:latest
    container_name: httpd
    ports:
      - "6100:80"
    volumes:
      - /opt/finance:/usr/local/apache2/htdocs
    
```

# Day 45: Resolve Dockerfile Issues
```
The Nautilus DevOps team is working to create new images per requirements shared by the development team. One of the team members is working to create a `Dockerfile` on `App Server 2` in `Stratos DC`. While working on it she ran into issues in which the docker build is failing and displaying errors. Look into the issue and fix it to build an image as per details mentioned below:

  

a. The `Dockerfile` is placed on `App Server 2` under `/opt/docker` directory.  
  

b. Fix the issues with this file and make sure it is able to build the image.  
  

c. Do not change base image, any other valid configuration within Dockerfile, or any of the data been used — for example, index.html.  
  

`Note:` Please note that once you click on `FINISH` button all the existing containers will be destroyed and new image will be built from your `Dockerfile`.
```

## Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Ejecutar el Dockerfile para ver el error
```
cd /opt/docker
docker buildx build --platform linux/amd64 -t issue .
```

```
Dockerfile:1
--------------------
   1 | >>> IMAGE httpd:2.4.43
   2 |     
   3 |     ADD sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
--------------------
ERROR: failed to build: failed to solve: dockerfile parse error on line 1: unknown instruction: IMAGE
```

Vamos a modificar **IMAGE** por **FROM**.
```
sudo vi Dockerfile
```

Finalmente el Dockerfile quedo
```
FROM httpd:2.4.43

RUN sed -i 's/Listen 80/Listen 8080/g' /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/
```

# Day 46: Deploy an App on Docker Containers
```
The Nautilus Application development team recently finished development of one of the apps that they want to deploy on a containerized platform. The Nautilus Application development and DevOps teams met to discuss some of the basic pre-requisites and requirements to complete the deployment. The team wants to test the deployment on one of the app servers before going live and set up a complete containerized stack using a docker compose fie. Below are the details of the task:  
  

  

1. On `App Server 2` in `Stratos Datacenter` create a docker compose file `/opt/itadmin/docker-compose.yml` (should be named exactly).  
      
    
2. The compose should deploy two services (web and DB), and each service should deploy a container as per details below:  
      
    

`For web service:`  
  

a. Container name must be `php_web`.  
  

b. Use image `php` with any `apache` tag. Check [here](https://hub.docker.com/_/php?tab=tags/) for more details.  
  

c. Map `php_web` container's port `80` with host port `5000`  
  

d. Map `php_web` container's `/var/www/html` volume with host volume `/var/www/html`.  
  

`For DB service:`  
  

a. Container name must be `mysql_web`.  
  

b. Use image `mariadb` with any tag (preferably `latest`). Check [here](https://hub.docker.com/_/mariadb?tab=tags/) for more details.  
  

c. Map `mysql_web` container's port `3306` with host port `3306`  
  

d. Map `mysql_web` container's `/var/lib/mysql` volume with host volume `/var/lib/mysql`.  
  

e. Set MYSQL_DATABASE=`database_web` and use any custom user ( except root ) with some complex password for DB connections.  
  

3. After running docker-compose up you can access the app with curl command `curl <server-ip or hostname>:5000/`  
      
    

For more details check [here](https://hub.docker.com/_/mariadb?tab=description/).  
  

`Note:` Once you click on `FINISH` button, all currently running/stopped containers will be destroyed and stack will be deployed again using your compose file.
```

## Ingresar al servidor
```
ssh steve@172.16.238.11
```

## Crear fichero
```
sudo touch /opt/itadmin/docker-compose.yml
```

### Configurar primer servicio
```
services:
  web:
    image: php:8.1-apache
    container_name: php_web
    ports:
      - 5000:80
    volumens:
      - /var/www/html:/var/www/html
```

### Configurar segundo servicio
```
  db:
    image: mariadb:latest
    container_name: mysql_web
    ports:
      - 3306:3306
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      - MARIADB_DATABASE:database_web
      - MARIADB_USER:simon
      - MARIADB_PASSWORD:D4n3r15_T4rg3r14n
```

### docker-compose.yml

```
sudo vi /opt/itadmin/docker-compose.yml
cd /opt/itadmin
```
	
```
services:
  web:
    image: php:8.1-apache
    container_name: php_web
    ports:
      - "5000:80"
    volumes:
      - /var/www/html:/var/www/html
    depends_on:
      - db
        
  db:
    image: mariadb:latest
    container_name: mysql_web
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MARIADB_ROOT_PASSWORD: hola_123
      MARIADB_DATABASE: database_web
      MARIADB_USER: simon
      MARIADB_PASSWORD: D4n3r15_T4rg3r14n

```

### Iniciar servicios
```
sudo docker compose up -d
```

# Day 47: Docker Python App
```
A python app needed to be Dockerized, and then it needs to be deployed on `App Server 3`. We have already copied a `requirements.txt` file (having the app dependencies) under `/python_app/src/` directory on `App Server 3`. Further complete this task as per details mentioned below:  
  

  

1. Create a `Dockerfile` under `/python_app` directory:
    
    - Use any `python` image as the base image.
    - Install the dependencies using `requirements.txt` file.
    - Expose the port `5002`.
    - Run the `server.py` script using `CMD`.  
          
        
2. Build an image named `nautilus/python-app` using this Dockerfile.  
      
    
3. Once image is built, create a container named `pythonapp_nautilus`:
    
    - Map port `5002` of the container to the host port `8096`.  
          
        
4. Once deployed, you can test the app using `curl` command on `App Server 3`.  
      
    

```sh
curl http://localhost:8096/
```

## Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Ir al directorio
```
cd /python_app
```

## Dockerfile
```
sudo vi Dockerfile
```

```
FROM python:3.10
WORKDIR /app
COPY . .
RUN cd src && \
    pip install --no-cache-dir -r requirements.txt
EXPOSE 5002
CMD ["python", "./src/server.py"]
```

## Construir imagen
```
docker buildx build --platform linux/amd64 -t nautilus/python-app .
```

## Construir container
```
docker run -d -p 8096:5002 --name pythonapp_nautilus nautilus/python-app
```

## Verificar 
```
curl http://localhost:8096
```

# Day 48: Deploy Pods in Kubernetes Cluster
```
The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:

  

1. Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag. Ensure to specify the tag as `httpd:latest`.
    
2. Set the `app` label to `httpd_app`, and name the container as `httpd-container`.
    

`Note`: The `kubectl` utility on `jump_host` is configured to operate with the Kubernetes cluster.
```

## Crear file pod.yml
```
vi pod.yml
```

## Crear pod
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
   - name: httpd-container
     image: httpd:latest
```

## Inicializar pod
```
kubectl apply -f pod.yml
```

## Verificar la cracion del POD
```
kubectl get pod
```

# Day 49: Deploy Applications with Kubernetes Deployments
```
The Nautilus DevOps team is delving into Kubernetes for app management. One team member needs to create a deployment following these details:


Create a deployment named nginx to deploy the application nginx using the image nginx:latest (ensure to specify the tag)

Note: The kubectl utility on jump_host is set up to interact with the Kubernetes cluster.
```

## Crear file deploy.yml
```
vi deploy.yml
```

## Agregar las especificaciones
```
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx
spec:
  selector:   
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

## Inicializar deployment
```
kubectl apply -f deploy.yml
```


# Day 50: Set Resource Limits in Kubernetes Pods
```
The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints. To address this, they plan to set limits on resource utilization. Here are the details:


Create a pod named httpd-pod with a container named httpd-container. Use the httpd image with the latest tag (specify as httpd:latest). Set the following resource limits:

Requests: Memory: 15Mi, CPU: 100m

Limits: Memory: 20Mi, CPU: 100m

Note: The kubectl utility on jump_host is configured to operate with the Kubernetes cluster.
```

## Crear file pod.yml
```
vi pod.yml
```

## Asignando recursos
```
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd_app
spec:
  containers:
   - name: httpd-container
     image: httpd:latest
     resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
```

## Inicializar pod
```
kubectl apply -f pod.yml
```

## Verificar
```
kubectl get pod
kubectl describe pod httpd-pod
```



