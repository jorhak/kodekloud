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
