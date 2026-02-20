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
9. 




