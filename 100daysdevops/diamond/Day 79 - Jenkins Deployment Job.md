```
The Nautilus development team had a meeting with the DevOps team where they discussed automating the deployment of one of their apps using Jenkins (the one in `Stratos Datacenter`). They want to auto deploy the new changes in case any developer pushes to the repository. As per the requirements mentioned below configure the required Jenkins job.  
  

  

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and `Adm!n321` password.  
  

Similarly, you can access the `Gitea UI` using `Gitea` button, username and password for Git is `sarah` and `Sarah_pass123` respectively. Under user `sarah` you will find a repository named `web` that is already cloned on the `Storage server` under sarah's home. `sarah` is a developer who is working on this repository.  
  

1. Install `httpd` (whatever version is available in the yum repo by default) and configure it to serve on port `8080` on All app servers. You can make it part of your Jenkins job or you can do this step manually on all app servers.  
  

2. Create a Jenkins job named `nautilus-app-deployment` and configure it in a way so that if anyone pushes any new change to the origin repository in `master` branch, the job should auto build and deploy the latest code on the `Storage server` under `/var/www/html` directory. Since `/var/www/html` on `Storage server` is shared among all apps.  
Before deployment, ensure that the ownership of the `/var/www/html` directory is set to user `sarah`, so that Jenkins can successfully deploy files to that directory.  
  

3. SSH into `Storage Server` using `sarah` user credentials mentioned above. Under sarah user's home you will find a cloned Git repository named `web`. Under this repository there is an `index.html` file, update its content to `Welcome to the xFusionCorp Industries`, then push the changes to the `origin` into `master` branch. This push must trigger your Jenkins job and the latest changes must be deployed on the servers, also make sure it deploys the entire repository content not only `index.html` file.  
  

Click on the `App` button on the top bar to access the app, you should be able to see the latest changes you deployed. Please make sure the required content is loading on the main URL `https://<LBR-URL>` i.e there should not be any sub-directory like `https://<LBR-URL>/web` etc.  
  

`Note:`  
1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also some times Jenkins UI gets stuck when Jenkins service restarts in the back end so in such case please make sure to refresh the UI page.  
  

2. Make sure Jenkins job passes even on repetitive runs as validation may try to build the job multiple times.  
  

3. Deployment related tasks should be done by `sudo` user on the destination server to avoid any permission issues so make sure to configure your Jenkins job accordingly.  
  

4. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

1. Instalar **APACHE**
- Ingresar a servidores
```
ssh tony@172.16.238.10
ssh steve@172.16.238.11
ssh banner@172.16.238.12
```

- Instalar **Apache**
```
sudo yum install -y httpd
```

- Configurar puerto
```
sudo vi /etc/httpd/conf/httpd.conf
```

Y cambiamos la linea 47 por:
```
Listen 8080
```

Reiniciamos el servicio
```
sudo systemctl restart httpd
```

2. Actualizar e instalar **Plugins**
- Ingresar a **Jenkins UI**
	 - Actualizar **Plugins**
	     **Dashboard>Manage Jenkins>Plugins>Updates** marcamos todos los plugins y click en **Update**.
	     
	 - Instalar **Plugins**
	     **Dashboard>Manage Jenkins>Plugins>Available plugins** en la barra de busqueda buscamos:
	     - SSH
	     - Pipeline
	     - Pipeline: GitHub Groovy Libraries
	     Y presionamos sobre **Install**

- Agregar credenciales
  **Dashboard>Manage Jenkins>Credentials>(global)** click sobre **Add Credentials** y llenamos el formulario con el usuario de _Gitea_.

- Configurar remote host
  **Dashboard>Manage Jenkins>System** en la seccion de **SSH remote hosts** damos click en **Add**, agregamos los datos del servidor **Storage server** (host, puerto, credentials este debe ser el de _sarah_), finalmente click en **Save**.

   ESTO NO ES NECESARIO!!!!!

3. Crear **Pipeline**
   Ingresamos a _Gitea_ y agregamos un nuevo fichero llamado _Jenkinsfile_:
```
pipeline {
    agent any
    stages{
        stage('Deploy') {
            steps{
	            sh '''
                    sshpass -p 'Sarah_pass123' ssh -o StrictHostKeyChecking=no sarah@172.16.238.15 "
                        cd /var/www/html && \
                        if [ -d .git ]; then
                            echo 'Actualizando repositorio...' && \
                            git pull origin master
                        else
                            echo 'Clonando por primera vez...' && \
                            git clone http://git.stratos.xfusioncorp.com/sarah/web_app.git .
                        fi
                    "
                '''
            }
        }
    }
}
```

5. Ingresar al servidor **Storage server** como _natasha_
```
ssh natasha@172.16.238.15
```

6. Dar permiso de escritura a /var/www/html
```
cd /var/www/
sudo chmod o+w html
```

7. Ingresar al servidor **Storage server** como _sarah_
```
ssh sarah@172.16.238.15
```

8. Descargar ultima version del repositorio
```
cd /var/www/html
git config --global --add safe.directory /var/www/html
git pull
```

9. Modificar el **index.html** desde _Gitea_
```
Welcome to the xFusionCorp Industries
```

10. Crear **Pipeline**
     **Dashboard>New Item** llenamos el formulario con el nombre propuesto y seleccionamos **Pipeline**, luego click sobre el boton **OK**.

     Nos dirigimos a la seccion **Pipeline**, en _Definition_ seleccionamos **Pipeline script from SCM**. En _SCM_ seleccionamos **Git**. En _Repository URL_ colocamos https://80-port-fl2k3ta4bbys32lf.labs.kodekloud.com/sarah/web y no http://git.stratos.xfusioncorp.com/sarah/web.git. En _Credentials_ seleccionamos a **sarah**. Todo lo demas lo dejamos tal como esta. Click en **Save**.
     
11. Ejecutar **Pipeline**
     **Dashboard>nautilus-app-deployment** click en el boton **Build Now**.

12. Verificar 
     - Primero vamos por el navegador web damos click sobre **App** en la plataforma de Kodekloud
     - Desde la terminal
```
curl 172.16.238.10:8080
curl 172.16.238.11:8080
curl 172.16.238.12:8080
```
