```
The DevOps team was looking for a solution where they want to restart Apache service on all app servers if the deployment goes fine on these servers in Stratos Datacenter. After having a discussion, they came up with a solution to use Jenkins chained builds so that they can use a downstream job for services which should only be triggered by the deployment job. So as per the requirements mentioned below configure the required Jenkins jobs.



Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and Adm!n321 password.


Similarly you can access Gitea UI on port 8090 and username and password for Git is sarah and Sarah_pass123 respectively. Under user sarah you will find a repository named web.


Apache is already installed and configured on all app server so no changes are needed there. The doc root /var/www/html on all these app servers is shared among the Storage server under /var/www/html directory.


1. Create a Jenkins job named nautilus-app-deployment and configure it to pull change from the master branch of web repository on Storage server under /var/www/html directory, which is already a local git repository tracking the origin web repository. Since /var/www/html on Storage server is a shared volume so changes should auto reflect on all apps.


2. Create another Jenkins job named manage-services and make it a downstream job for nautilus-app-deployment job. Things to take care about this job are:


a. This job should restart httpd service on all app servers.

b. Trigger this job only if the upstream job i.e nautilus-app-deployment is stable.


LB server is already configured. Click on the App button on the top bar to access the app. You should be able to see the latest changes you made. Please make sure the required content is loading on the main URL https://<LBR-URL> i.e there should not be a sub-directory like https://<LBR-URL>/web etc.


Note:


1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also some times Jenkins UI gets stuck when Jenkins service restarts in the back end so in such case please make sure to refresh the UI page.


2. Make sure Jenkins job passes even on repetitive runs as validation may try to build the job multiple times.


3. Deployment related tasks should be done by sudo user on the destination server to avoid any permission issues so make sure to configure your Jenkins job accordingly.


4. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

1. Configurar **Jenkins UI**:
	Ingresamos con los credenciales provistos.
- Actualizar _Plugins_:
		**Dashboard>Manage Jenkins>Plugins>Updates** marcamos todos los plugins y click en **Update**.
- Instalar **Plugins**
	     **Dashboard>Manage Jenkins>Plugins>Available plugins** en la barra de busqueda buscamos:
	     - SSH
	     - Pipeline
	     - Pipeline: GitHub Groovy Libraries
	     Y presionamos sobre **Install**

2. Agregar **Credenciales**
	**Dashboard>Manage Jenkins>Credentials>(global)** click sobre **Add Credentials** y llenamos el formulario con el usuario de _Gitea_.

3. Crear **Jenkins job** (nautilus-app-deployment)
	 **Dashboard>New Item** llenamos el formulario con el nombre propuesto y seleccionamos **Pipeline**, luego click sobre el boton **OK**.

     Nos dirigimos a la seccion **Pipeline**, en _Definition_ seleccionamos **Pipeline script from SCM**. En _SCM_ seleccionamos **Git**. En _Repository URL_ colocamos https://80-port-7jhn4uhf6nw3q5pj.labs.kodekloud.com/sarah/web y no http://git.stratos.xfusioncorp.com/sarah/web.git. En _Credentials_ seleccionamos a **sarah**. Todo lo demas lo dejamos tal como esta. Click en **Save**.

4. Crear **Jenkinsfile**
	Crear el fichero **Jenkinsfile** nos vamos a _Gitea_ e ingresamos con los credenciales provistos y damos click sobre el repositorio **sarah/web**, luego sobre **New File**
```Jenkinsfile
pipeline {
    agent any
    stages{
        stage('Pull') {
            steps{
	            sh '''
                    sshpass -p 'Bl@kW' ssh -o StrictHostKeyChecking=no natasha@172.16.238.15 "
                        cd /var/www/html && \
                        if [ -d .git ]; then
                            echo 'Actualizando repositorio...' && \
                            git pull origin master
                        else
                            echo 'Clonando por primera vez...' && \
                            git clone http://git.stratos.xfusioncorp.com/sarah/web.git .
                        fi
                    "
                '''
            }
        }
    }
}
```

En la parte inferior agregamos el commit y una descripcion del commit.
Finamente damos click sobre **Commit Changes**.

6. Crear **Jenkins Job** (manage-services)
	**Dashboard>New Item** llenamos el formulario con el nombre propuesto y seleccionamos **Freestyle project**, luego click sobre el boton **OK**.
	- Nos dirigimos a la seccion **Triggers**, marcamos _Build after other projects are built_ en **Project to watch** introducimos _nautilus-app-deployment_, luego marcamos _Trigger only if build is stable_.
	- Nos dirigimos a la seccion **Build Steps** en _Add build step_ marcamos **Execute shell**:
```
sshpass -p "Ir0nM@n" ssh -o 'StrictHostKeyChecking=no' -t tony@stapp01 'echo "Ir0nM@n" | sudo -S systemctl restart httpd'

sshpass -p "Am3ric@" ssh -o 'StrictHostKeyChecking=no' -t steve@stapp02 'echo "Am3ric@" | sudo -S systemctl restart httpd'

sshpass -p "BigGr33n" ssh -o 'StrictHostKeyChecking=no' -t banner@stapp03 'echo "BigGr33n" | sudo -S systemctl restart httpd'
```
		
Finalmente damos click sobre **Save**.
	
6. Ingresamos al servidor **App1** para ver su estado
```
ssh tony@172.16.238.10
```

```
cd /var/www/html
cat index.html
sudo systemctl status httpd
```

7. Ingresamos al servidor **Storage server**
```
ssh natasha@172.16.238.15
```

```
cd /var/www/html
cat index.html
ls -la
```

8. Ejecutar **Pipeline**
	**Dashboard>nautilus-app-development>Build Now**
	Debemos esperar a que se termine de ejecutar _manage-services_. Nos dirigemos a **Dashboard**, ambos items deben estar en verde.
