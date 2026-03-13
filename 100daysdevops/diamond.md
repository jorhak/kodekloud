# Day 76: Jenkins Project Security
```
The xFusionCorp Industries has recruited some new developers. There are already some existing jobs on Jenkins and two of these new developers need permissions to access those jobs. The development team has already shared those requirements with the DevOps team, so as per details mentioned below grant required permissions to the developers.  
  

  

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.  
  

1. There is an existing Jenkins job named `Packages`, there are also two existing Jenkins users named `sam` with password `sam@pass12345` and `rohan` with password `rohan@pass12345`.  
      
    
2. Grant permissions to these users to access `Packages` job as per details mentioned below:  
      
    
    a.) Make sure to select `Inherit permissions from parent ACL` under `inheritance strategy` for granting permissions to these users.  
      
    
    b.) Grant mentioned permissions to `sam` user : `build`, `configure` and `read`.  
      
    
    c.) Grant mentioned permissions to `rohan` user : `build`, `cancel`, `configure`, `read`, `update` and `tag`.  
      
    

`Note:`  
  

1. Please do not modify/alter any other existing job configuration.  
      
    
2. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.  
      
    
3. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

Luego de ingresar a **Jenkins** lo primero que vamos hacer sera actualizar los plugins y como segundo instalar los plugins que vamos a necesitar, para ello nos dirigimos a **Dashboard>Manage Jeninks>Plugins>Updates** y marcamos todos para que se actulice, luego nos dirigimos a **Dashboard>Manage Jenkins>Plugins>Available plugins** y en la barra de busqueda introducimos _Matrix Authorization Strategy, SSH_ y lo marcamos, asi respectivamente y reiniciamos **Jenkins**.

Ahora vamos a habilitar la autorizacion, nos dirigimos a **Dashboard>Manage Jenkins>Security**, buscamos el apartado _Authorization_, damos click y seleccionamos _Project based Matrix Authorization Strategy_ luego damos click en el boton **Save**.

Seguiendo los requerimientos vamos a configurar el **Job**, nos dirigimos **Dashboard>Packages>**, damos click sobre _Configure_, en **General** marcamos _Enable project-based security_ y debe estar seleccionado _Inhert permissions from parent ACL_.
Luego debemos agregar los usuarios _sam_ y _rohan_ con las caracteristicas requeridas. Para ello luego de haber seleccionado la estrategia damos click sobre el boton **Add user...**, introducimos el **User Id** y presionamos el boton **OK**, de la misma forma agregamos los usuarios pertinentes.
Una ves los usuarios estan agregados vamos a asignarles los permisos que son requeridos y finalmente damos click sobre el boton **Save**.

_NOTA_:
Comandos que estan en **Build Steps**
```
sshpass -p contra ssh -o 'StrictHostKeyChecking=no' -t tony@stapp01 'echo "contra" | sudo -S yum install git wget telnet net-tools zip -y'

sshpass -p contra ssh -o 'StrictHostKeyChecking=no' -t steve@stapp02 'echo "contra" | sudo -S yum install git wget telnet net-tools zip -y'

sshpass -p contra ssh -o 'StrictHostKeyChecking=no' -t banner@stapp03 'echo "contra" | sudo -S yum install git wget telnet net-tools zip -y'
```

# Day 77: Jenkins Deploy Pipeline
```
The development team of xFusionCorp Industries is working on to develop a new static website and they are planning to deploy the same on Nautilus App Servers using Jenkins pipeline. They have shared their requirements with the DevOps team and accordingly we need to create a Jenkins pipeline job. Please find below more details about the task:



Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and password Adm!n321.


Similarly, click on the Gitea button on the top bar to access the Gitea UI. Login using username sarah and password Sarah_pass123. There under user sarah you will find a repository named web_app that is already cloned on Storage server under /var/www/html. sarah is a developer who is working on this repository.


Add a slave node named Storage Server. It should be labeled as ststor01 and its remote root directory should be /var/www/html.


We have already cloned repository on Storage Server under /var/www/html.


Apache is already installed on all app Servers its running on port 8080.


Create a Jenkins pipeline job named datacenter-webapp-job (it must not be a Multibranch pipeline) and configure it to:


Deploy the code from web_app repository under /var/www/html on Storage Server, as this location is already mounted to the document root /var/www/html of app servers. The pipeline should have a single stage named Deploy ( which is case sensitive ) to accomplish the deployment.

LB server is already configured. You should be able to see the latest changes you made by clicking on the App button. Please make sure the required content is loading on the main URL https://<LBR-URL> i.e there should not be a sub-directory like https://<LBR-URL>/web_app etc.


Note:


You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.


For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

**OPCION 1**
Lo primero que vamos hacer sera actualizar los plugins y instalar los plugins necesarios: **SSH**, **SSH Build Agents**, **Pipeline**, **Pipeline: GitHub Groovy Libraries**.
Para ello nos dirigimos **Dashboard>Manage Jenkins>Plugins** en _Updates_ marcamos los plugins que vamos a actualizar, luego nos vamos a _Available plugins_ y en la barra de busqueda escribimos los plugins que mencionamos previamente.

Luego debemos agregar los credenciales del servidor **ststor01** y del usuario de **Gitea** (sarah). Para ello nos dirigimos **Dashboard>Manage Jenkins>Credentials**, damos click en _(global)_, luego sobre **Add Credentials**. _Kind_ debe estar en _Username with password_, rellenamos el formulario y presionamos **Create**.

Vamos a configurar el acceso remoto, para ello nos dirigimos **Dashboard>Manage Jenkins>System**, nos dirigimos al apartado _SSH remote hosts_ y damos click sobre **Add** y rellenamos el formulario con los datos correspondientes y damos click sobre **Save**.

Ingresar en el servidor esclavo:
```
ssh natasha@172.16.238.15
```

En el servidor escalvo debemos instalar **Java**:
```
sudo yum install -y java-21-openjdk.x86_64 # Instala java
sudo update-alternatives --display java # Muestra ruta de instalacion
java --version # Verifica instalacion de java
```

Dar permiso de escritura:
```
sudo chmod o+w -R /var/www/html
```

Agregar el **Nodo** para ello nos dirigimos a **Dashboard>Manage Jenkins>Nodes** y presionamos el boton **New Node**, le asignamos el nombre establecido y marcamos _Permanent Agent_, finalmente presionamos **Create**. Se nos abrira un formulario en donde debemos rellenar con los datos provistos y en _Launch method_ seleccionamos _Launch agents via SSH_, agregamos el host, credencial y en _Host Key Verification Strategy_ seleccionamos _Manually trusted key Verification Strategy_, finalmente damos click sobre **Save**.

Crear **Pipeline** nos dirigimos a **Dashboard**, damos click sobre _New Item_ le damos el nombre propuesto y seleccionamos **Pipeline** luego damos click sobre el boton **OK**. Luego se nos abrira un formulario, buscamos el apartado **Pipeline**, en _Definition_ seleccionamos _Pipeline script from SCM_, en _SCM_ seleccionamos **Git**, en _Repository URL_ agregamos la url del repositorio, en _Credentials_ le asignamos el credencial previamente creado para el acceso a **Gitea** (sarah), en _Script Path_ debemos tener **Jenkinsfile**.

Ingresamos a Gitea y agregamos el fichero **Jenkinsfile**:
```
pipeline {
    agent any
    stages{
        stage('Deploy') {
            steps {
                git branch: 'master',
                    credentialsId: 'sarah_gitea',
                    url: 'http://git.stratos.xfusioncorp.com/sarah/web_app.git'
	            sh "sshpass -p 'Bl@kW' scp -r -o StrictHostKeyChecking=no ./* natasha@172.16.238.15:/var/www/html"
            }
        }
    }
}
```

Verificar que los cambios se realizan, abrimos un navegador:
```
https://8091-port-4t6zkxingakha3at.labs.kodekloud.com
```

Realizamos cambios desde **Gitea**, volvemos a ejecutar el **Pipeline** y actualizamos el navegador para ver los cambios efectuados en _index.html_.

**OPCION 2**
Ingresar en el servidor esclavo:
```
ssh natasha@172.16.238.15
```

En el servidor escalvo debemos instalar java:
```
sudo yum install java-21-openjdk.x86_64 -y # Instala java
sudo update-alternatives --display java # Muestra ruta de instalacion
java --version # Verifica instalacion de java
```

Tambien debemos cambiar los permisos del directorio /var/www/html
```
sudo chmod o+w -R /var/www/html
```

Clonar repositorio:
```
cd /var/www/html
git clone http://git.stratos.xfusioncorp.com/sarah/web_app.git
cd web_app
git config --global user.email "jorhak@kodekloud.com"
git config --global user.name "Jorhak"
```

Crear fichero **Jenkinsfile** que va ejecutar **Pipeline**:
```
pipeline {
    agent any // Specifies where the pipeline will run (any available agent/node)
    stages {
        stage('Deploy') {
            steps {
                echo 'Deploying the application...' // Example deploy step
            }
        }
    }
}
```

Lo primero que vamos hacer sera actualizar los plugins y instalar los plugins necesarios: **Pipeline**, **SSH**, **SSH Build Agent, Pipeline, Pipeline: GitHub Groovy Libraries**.

Luego debemos agregar los credenciales del servidor **stapp01**. Para ello nos dirigimos **Dashboard>Manage Jenkins>Credentials**, damos click en _(global)_, luego sobre **Add Credentials**. _Kind_ debe estar en _Username with password_, rellenamos el formulario y presionamos **Create**.

Vamos a configurar el acceso remoto, para ello nos dirigimos **Dashboard>Manage Jenkins>System**, nos dirigimos al apartado _SSH remote hosts_ y damos click sobre **Add** y rellenamos el formulario con los datos correspondientes y damos click sobre **Save**.

Agregar el **Nodo** para ello nos dirigimos a **Dashboard>Manage Jenkins>Nodes** y presionamos el boton **New Node**, le asignamos el nombre establecido y marcamos _Permanent Agent_, finalmente presionamos **Create**. Se nos abrira un formulario en donde debemos rellenar con los datos provistos y en _Launch method_ seleccionamos _Launch agents via SSH_, agregamos el host, credencial y en _Host Key Verification Strategy_ seleccionamos _Manually trusted key Verification Strategy_, finalmente damos click sobre **Save**.

Ahora vamos a crear el **Pipeline** para ello nos dirigimos **Dashboard>New Item** le damos el nombre asignando y seleccionamos **Pipeline** y luego click en **OK**.
Nos abrira otro formulario en le apartado **Pipeline** en _Definition_ seleccionamos **Pipeline script from SCM** rellenamos el formulario y para finalizar damos click en **Save**.

Crear **Pipeline** nos dirigimos a **Dashboard**, damos click sobre _New Item_

Verificar que los cambios se realizan, abrimos un navegador:
```
https://8091-port-4t6zkxingakha3at.labs.kodekloud.com/web_app/
```

# Day 78: Jenkins Conditional Pipeline
```
The development team of xFusionCorp Industries is working on to develop a new static website and they are planning to deploy the same on Nautilus App Servers using Jenkins pipeline. They have shared their requirements with the DevOps team and accordingly we need to create a Jenkins pipeline job. Please find below more details about the task:  
  

  

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.  
  

Similarly, click on the `Gitea` button on the top bar to access the Gitea UI. Login using username `sarah` and password `Sarah_pass123`. There under user `sarah` you will find a repository named `web_app` that is already cloned on `Storage server` under `/var/www/html`. sarah is a developer who is working on this repository.  
  

1. Add a slave node named `Storage Server`. It should be labeled as `ststor01` and its remote root directory should be `/var/www/html`.  
      
    
2. We have already cloned repository on `Storage Server` under `/var/www/html`.  
      
    
3. Apache is already installed on all app Servers its running on port `8080`.  
      
    
4. Create a Jenkins pipeline job named `nautilus-webapp-job` (it must not be a `Multibranch pipeline`) and configure it to:  
      
    
    - Add a string parameter named `BRANCH`.  
          
        
    - It should conditionally deploy the code from `web_app` repository under `/var/www/html` on `Storage Server`, as this location is already mounted to the document root `/var/www/html` of app servers. The pipeline should have a single stage named `Deploy` ( which is case sensitive ) to accomplish the deployment.  
          
        
    - The pipeline should be conditional, if the value `master` is passed to the `BRANCH` parameter then it must deploy the `master` branch, on the other hand if the value `feature` is passed to the `BRANCH` parameter then it must deploy the `feature` branch.  
          
        

LB server is already configured. You should be able to see the latest changes you made by clicking on the `App` button. Please make sure the required content is loading on the main URL `https://<LBR-URL>` i.e there should not be a sub-directory like `https://<LBR-URL>/web_app` etc.  
  

`Note:`  
  

1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.  
      
    
2. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

1. Ingresar a **Jenkins UI**:
	- Actualizar los plugins
     **Dashboard>Manage Jenkins>Plugins>Updates** los marcamos a todos y damos click sobre **Update**.
    - Instalamos los plugins necesarios
      **Dashboard>Manage Jenkins>Plugins>Available plugins**.
      En la barra de busqueda escribimos:
         - Pipeline
         - Pipeline: GitHub Groovy Libraries
         - SSH
         - SSH Build Agents
        Luego de que lo marcamos click en **Install**
        
2. Ingresar al servidor esclavo:
```
ssh natasha@172.16.238.15
```

3. Instalar java en servidor esclavo
```
sudo yum install -y java-21-openjdk.x86_64 # Instala java
sudo update-alternatives --display java # Muestra ruta de instalacion
java --version # Verifica instalacion de java
```
    
    Dar permiso de escritura:
```
sudo chmod o+w -R /var/www/html
```

4. Agregar credenciales
   Debemos agregar las credenciales de:
     - Storage server
     - Gitea
    **Dashboard>Manage Jenkins>Credentials** damos click sobre _(global)_, click en **Add Credentials**
    
5. Agregar Remote Host
     **Dashboard>Manage Jenkins>System**. En la seccion **SSH remote hosts** click en **Add**. Rellenamos el formulario con los datos provistos. En este caso necesitamos los datos de **Storage Server**.
     Finalmente click en **Save**.
     
6. Agregar nodo esclavo
     **Dashboard>Manage Jenkins>Nodes** click sobre **New Node**, luego le asignamos el nombre y marcamos como _Type_: _Permanent Agent_, click en **Create**.
     Llenamos los campos con los datos propuestos de **Storage Server**.
     En launch method seleccionamos _Launch agents via SSH_.
     En host colocamos la **IP** del servidor.
     En Credentials seleccionamos _natasha....._.
     En Host Key Verification Strategy seleccionamos _Manually trusted key Verification Strategy_.
     Finalmente click en **Save**.
7. Ingresar a Gitea
     Creamos un fichero de nombre **Jenkinsfile**:
```
pipeline {
    agent {label 'ststor01'}
    stages{
        stage('Deploy') {
            steps {
                script { 
	                def targetBranch = ""
	                    if (params.BRANCH == 'master') {
	                        targetBranch = 'master'
	                    } else if (params.BRANCH == 'feature') {
	                        targetBranch = 'feature'
	                    } else {
	                        error "Error: Solo se permite 'master' o 'feature'. Valor recibido: ${params.BRANCH}"
	                    }

	                    echo "Iniciando despliegue de la rama: ${targetBranch}"
                
	                git branch: targetBranch,
	                    credentialsId: 'gitea',
	                    url: 'http://git.stratos.xfusioncorp.com/sarah/web_app.git'
                }
	            sh "sshpass -p 'Bl@kW' scp -r -o StrictHostKeyChecking=no ./* natasha@172.16.238.15:/var/www/html"
            }
        }
    }
}
```

    Guardamos el fichero Jenkinsfile. este fichero debe estar en ambas ramas.
    
8. Crear **Pipeline**
   **Dashboard** click sobre **New Item**, le asignamos el nombre, click sobre _Pipeline_, luego sobre **OK**.
     - Agregar parametro
         En la seccion **General** marcamos _This project is parametrized_, le asignamos el nombre, valor por defecto y descripcion.
     - Pileline
         En la seccion Pipeline:
         **Defenition** seleccionamos _Pipeline script from SCM_
         **SCM** seleccionamos _Git_
         **Repository URL** agregamos la url del repositorio se debe agregar https://80-port-g7zmbyeuelo67qtk.labs.kodekloud.com/sarah/web_app y no http://git.stratos.xfusioncorp.com/sarah/web_app.git porque de lo contrario no funciona.
         **Credentials** seleccionamos el credencial del repositorio
         **Branches to build** agregamos una rama _feature_.
         Finalmente click en **Save**.

# Day 79: Jenkins Deployment Job
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
     
12. Ejecutar **Pipeline**
     **Dashboard>nautilus-app-deployment** click en el boton **Build Now**.

13. Verificar 
     - Primero vamos por el navegador web damos click sobre **App** en la plataforma de Kodekloud
     - Desde la terminal
```
curl 172.16.238.10:8080
curl 172.16.238.11:8080
curl 172.16.238.12:8080
```

# Day 80: Jenkins Chained Builds
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

# Day 81: Jenkins Multistage Pipeline
```
The development team of xFusionCorp Industries is working on to develop a new static website and they are planning to deploy the same on Nautilus App Servers using Jenkins pipeline. They have shared their requirements with the DevOps team and accordingly we need to create a Jenkins pipeline job. Please find below more details about the task:  
  

  

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.  
  

Similarly, click on the `Gitea` button on the top bar to access the Gitea UI. Login using username `sarah` and password `Sarah_pass123`.  
  

There is a repository named `sarah/web` in Gitea that is already cloned on `Storage server` under `/var/www/html` directory.  
  

1. Update the content of the file `index.html` under the same repository to `Welcome to xFusionCorp Industries` and push the changes to the origin into the `master` branch.  
      
    
2. Apache is already installed on all app Servers its running on port `8080`.  
      
    
3. Create a Jenkins pipeline job named `deploy-job` (it must not be a `Multibranch pipeline` job) and pipeline should have two stages `Deploy` and `Test` ( names are case sensitive ). Configure these stages as per details mentioned below.  
      
    
    a. The `Deploy` stage should deploy the code from `web` repository under `/var/www/html` on the `Storage Server`, as this location is already mounted to the document root `/var/www/html` of all app servers.  
      
    
    b. The `Test` stage should just test if the app is working fine and website is accessible. Its up to you how you design this stage to test it out, you can simply add a `curl` command as well to run a curl against the LBR URL (`http://stlb01:8091`) to see if the website is working or not. Make sure this stage fails in case the website/app is not working or if the `Deploy` stage fails.  
      
    

Click on the `App` button on the top bar to see the latest changes you deployed. Please make sure the required content is loading on the main URL `http://stlb01:8091` i.e there should not be a sub-directory like `http://stlb01:8091/web` etc.  
  

`Note:`  
  

1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.  
      
    
2. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
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

3. Ingresar al servidor **Storage Server**:
```
ssh natasha@172.16.238.15
```

- Verificar contenido
```
cd /var/www/html
cat index.html
```

4. Actualizar **index.html**:
	Ingresar a Gitea con las credenciales provistas, ingresar al respositorio **sarah/web**, y editar el fichero _index.html_

```
Welcome to xFusionCorp Industries
```

Agregar un commit y una descripcion, click en **Commit Changes**.

5. Crear **Pipeline** (deploy-job)
	**Dashboard>New Item** llenamos el formulario con el nombre propuesto y seleccionamos **Pipeline**, luego click sobre el boton **OK**.

     Nos dirigimos a la seccion **Pipeline**, en _Definition_ seleccionamos **Pipeline script from SCM**. En _SCM_ seleccionamos **Git**. En _Repository URL_ colocamos https://80-port-dsb4s67vcvq7xx4y.labs.kodekloud.com/sarah/web y no http://git.stratos.xfusioncorp.com/sarah/web.git. En _Credentials_ seleccionamos a **sarah**. Todo lo demas lo dejamos tal como esta. Click en **Save**.

6. Crear **Jenkinsfile**
	Crear el fichero **Jenkinsfile** nos vamos a _Gitea_ e ingresamos con los credenciales provistos y damos click sobre el repositorio **sarah/web**, luego sobre **New File**
```Jenkinsfile
pipeline {
    agent any
    stages{
        stage('Deploy') {
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
        
        stage('Test'){
	        steps{
		        script{
		           echo "Verificar LoadBalancer"
			       def response = sh(script: "curl -fsI http://stlb01:8091", returnStatus: true)
			       if (response!=0){
				       error "Test fallido: http://stlb01:8091 no es accesible."
			       } else {
			          echo "Test exitoso!!!!: http://stlb01:8091 accesible."
			       }
		        }
	        }
        }
        
    }
    post{
	        failure {
		        echo "Pipeline fallo, ver los logs de Deploy y Test"
	        }
        }
}
```

Asignamos el nombre de **Jenkinsfile**, en la parte inferior agregamos un commit y una descripcion, click en **Commit Changes**.

7. Ejecutar
	**Dashboard>deploy-job>Build now**

# Day 82: Create Ansible Inventory for App Server Testing
```
The Nautilus DevOps team is testing Ansible playbooks on various servers within their stack. They've placed some playbooks under /home/thor/playbook/ directory on the jump host and now intend to test them on app server 3 in Stratos DC. However, an inventory file needs creation for Ansible to connect to the respective app. Here are the requirements:


a. Create an ini type Ansible inventory file /home/thor/playbook/inventory on jump host.


b. Include App Server 3 in this inventory along with necessary variables for proper functionality.


c. Ensure the inventory hostname corresponds to the server name as per the wiki, for example stapp01 for app server 1 in Stratos DC.


Note: Validation will execute the playbook using the command ansible-playbook -i inventory playbook.yml. Ensure the playbook functions properly without any extra arguments.
```

playbook.yml
```
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Install httpd package
      yum:
        name: httpd
        state: installed

    - name: Start service httpd
      service:
        name: httpd
        state: started
```

Ingresar por ssh a stapp03
```
ssh banner@stapp03
cat /etc/hosts
```

Abrimos otra terminal capturamos su IP y Host de stapp03. Y lo copiamos en /etc/hosts de JUMP-HOST
```
nano /etc/hosts
```

```
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.244.97.152   jump-host

# Entries added by HostAliases.
10.0.15.5       docker-registry-mirror.kodekloud.com
10.244.49.93    stapp03
```

1. Ir a directorio
```
cd /home/thor/playbook/
```

2. Crear inventario
```
nano inventory
```

```
[server]
stapp03 ansible_host=10.244.49.93 ansible_user=banner ansible_password=BigGr33n
```

3. Verificar inventario
```
ansible-inventory -i inventory --list
```

4. Ping a server
```
ansible server -m ping -i inventory
```

5. Ejecutar playbook
```
ansible-playbook -i inventory playbook.yml
```
