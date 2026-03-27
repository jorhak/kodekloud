```
The Nautilus DevOps team has installed and configured new Jenkins server in Stratos DC which they will use for CI/CD and for some automation tasks. There is a requirement to add all app servers as slave nodes in Jenkins so that they can perform tasks on these servers using Jenkins. Find below more details and accomplish the task accordingly.  
  

  

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.  
  
  
1. Add all app servers as SSH build agent/slave nodes in Jenkins. Slave node name for `app server 1`, `app server 2` and `app server 3` must be `App_server_1`, `App_server_2`, `App_server_3` respectively.  
  
  
2. Add labels as below:  
  
  
`App_server_1 : stapp01`  
  
`App_server_2 : stapp02`  
  
`App_server_3 : stapp03`  
  
  
3. Remote root directory for `App_server_1` must be `/home/tony/jenkins`, for `App_server_2` must be `/home/steve/jenkins` and for `App_server_3` must be `/home/banner/jenkins`.  
  
  
4. Make sure slave nodes are online and working properly.  
  

`Note:`  

1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.  
  
  
2. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

Instalar en cada servidor java version 21 (openjdk-21) y capturar su path.
```
ssh tony@172.16.238.10
ssh steve@172.16.238.11
ssh banner@172.16.238.12
```

```
sudo yum install java-21-openjdk.x86_64 -y # Instala java
sudo update-alternatives --display java # Muestra ruta de instalacion
java --version # Verifica instalacion de java
```

Luego de ingresar a **Jenkins** actualizamos los plugins. Nos dirigimos a **Dashboard>Manage Jenkins>Plugins**. En _Updates_ actualizamos los plugins. Y en _Available plugins_, en la barra de busqueda colocamos **SSH y SSH Build Agents** y lo agregamos.

Segundo paso sera crear los credenciales para el acceso. Nos dirigimos a **Dashboard>Manage Jenkins>Credentials** ahora damos click en _global_ y luego en el boton **Add Credentials**. En _Kind_ seleccionamos **Username with password** y rellenamos el formulario con los datos correspondientes de cada servidor.

Tercer paso configurar la conexion de SSH, para ello nos dirigimos a **Dashboard>Manage Jenkins>System** y nos dirigimos al apartado _SSH remote hosts_ y damos click en **Add** y agregamos los tres servidores por ulitimo damos click en **Save**.

Como cuarto paso vamos a agregar los nodos, nos dirigimos a **Dashboard>Manage Jenkins>Nodes**, luego damos click en _New Node_, rellenamos el formulario con el nombre propuesto y marcamos en _Type_ marcamos _Permanent Agent_ y por ultimo damos click sobre **Create**, completamos el formulario con los datos propuestos, en **Launch method** seleccionamos _Launch agents via SSH_ y en **Host Key Verification Strategy** seleccionamos _Manually trusted key Verification Strategy_ y finalmente damos click en **Save**.
