# 1 Set Up Git Repository on Storage Server
```
The Nautilus development team has provided requirements to the DevOps team for a new application development project, specifically requesting the establishment of a Git repository. Follow the instructions below to create the Git repository on the `Storage server` in the Stratos DC:  
  

  

1. Utilize `yum` to install the `git` package on the `Storage Server`.  
      
    
2. Create a bare repository named `/opt/beta.git` (ensure exact name usage).
```

1. Instalar Git
- Ingresar al servidor
```
ssh natasha@ststor01
```

- Intalar
```
sudo yum update -y && sudo yum install git -y
```

- Verificar instalacion
```
git -v
```

2. Crear repositorio
```
cd /opt
sudo git init --bare demo.git
```

 git bare repository '/opt/apps.git' not found on Storage Server or its not a bare repository

# 2 Clone Git Repository on Storage Server
```
The DevOps team established a new Git repository last week, which remains unused at present. However, the Nautilus application development team now requires a copy of this repository on the `Storage Server` in the Stratos DC. Follow the provided details to clone the repository:  
  

  

1. The repository to be cloned is located at `/opt/official.git`  
      
    
2. Clone this Git repository to the `/usr/src/kodekloudrepos` directory. Perform this task using the natasha user, and ensure that no modifications are made to the repository or existing directories, such as changing permissions or making unauthorized alterations.
```

1. Ingresar al servidor
```
ssh natasha@ststor01
```

2. Ir al directorio
```
cd /usr/src/kodekloudrepos
```

3. Clonar repositorio
```
git clone /opt/official.git
```
3. Verificar
```
ls
```

# 3 Fork a Git Repository
```
There is a Git server utilized by the Nautilus project teams. Recently, a new developer named Jon joined the team and needs to begin working on a project. To begin, he must fork an existing Git repository. Follow the steps below:



Click on the Gitea UI button located on the top bar to access the Gitea page.


Login to Gitea server using username jon and password Jon_pass123.


Once logged in, locate the Git repository named sarah/story-blog and fork it under the jon user.


Note: For tasks requiring web UI changes, screenshots are necessary for review purposes. Additionally, consider utilizing screen recording software such as loom.com to record and share your task completion process.
```

1. Ingresar a **Gitea**:
	Lo que hacemos es ingresar presionando el boton **Gitea UI** y luego click en **Sign In** por ultimo ingresamos el **Username** y **Password** proporcionados.
2. Buscar **usuario**:
	Despues de logearnos presionamos en **Explore users** y buscamos al usuario **sarah** damos click. Luego nos va a redireccionar el repositorio de **sarah** donde damos click sobre el repositorio **store-blog**.
3. Fork repository
	Finalmente damos click sobre **Fork** que se encuentra en la parte superior derecha, se nos abrira un formulario lo dejamos tal cual y presionamos sobre **Fork Repository**.

# 4 Update Git Repository with Sample HTML File
```
The Nautilus development team has initiated a new project development, establishing various Git repositories to manage each project's source code. Recently, a repository named /opt/demo.git was created. The team has provided a sample index.html file located on the jump host under the /tmp directory. This repository has been cloned to /usr/src/kodekloudrepos on the storage server in the Stratos DC.



Copy the sample index.html file from the jump host to the storage server placing it within the cloned repository at /usr/src/kodekloudrepos/demo.

Add and commit the file to the repository.

Push the changes to the master branch.
```

1. Ingresar a servidor
	Ingresamos al servidor para verificar el directorio del repositorio
```
ssh natasha@ststor01
```

```
cd /usr/src/kodekloudrepos && ls -la
```

2. Copiar _index.html_ de **jump host** a **storage server**
	Abrimos otra terminal y ejecutamos
```
scp /tmp/index.html natasha@ststor01:/home/natasha
```

3. Mover _index.html_
	Volvemos a la terminal previa y ejecutamos
```
sudo mv index.html /usr/src/kodekloudrepos/demo && cd /usr/src/kodekloudrepos/demo
```

4. Agregar y commit
```
git config --global --add safe.directory /usr/src/kodekloudrepos/demo
```

```
sudo git add index.html
sudo git commit -m "Add index.html"
sudo git push
```

# 5 Delete Git Branch
```
The Nautilus developers are engaged in active development on one of the project repositories located at /usr/src/kodekloudrepos/media. During testing, several test branches were created, and now they require cleanup. Here are the requirements provided to the DevOps team:



On the Storage server in Stratos DC, delete a branch named xfusioncorp_media from the /usr/src/kodekloudrepos/media Git repository.
```

1. Ingresar al servidor
```
ssh natasha@ststor01
```

2. Eliminar branch
```
cd /usr/src/kodekloudrepos/media
git config --global --add safe.directory /usr/src/kodekloudrepos/media
git branch
sudo git checkout master
sudo git branch -d xfusioncorp_media
```

3. 