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
