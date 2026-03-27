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
