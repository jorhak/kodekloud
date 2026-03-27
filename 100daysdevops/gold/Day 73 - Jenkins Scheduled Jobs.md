```
The devops team of xFusionCorp Industries is working on to setup centralised logging management system to maintain and analyse server logs easily. Since it will take some time to implement, they wanted to gather some server logs on a regular basis. At least one of the app servers is having issues with the Apache server. The team needs Apache logs so that they can identify and troubleshoot the issues easily if they arise. So they decided to create a Jenkins job to collect logs from the server. Please create/configure a Jenkins job as per details mentioned below:



Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and password Adm!n321

1. Create a Jenkins jobs named copy-logs.

2. Configure it to periodically build every 5 minutes to copy the Apache logs (both access_log and error_logs) from App Server 3 (from default logs location) to location /usr/src/finance on Storage Server.

Note:

1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case please make sure to refresh the UI page.

2. Please make sure to define you cron expression like this */10 * * * * (this is just an example to run job every 10 minutes).

3. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

Luego de ingresar a **Jenkins** damos click en **Create job** y le asignamos el nombre provisto y seleccionar **Freestyle project**.


Despues nos vamos a actualizar los _plugins_, seguido de eso vamos a instalar los _plugins_: **SSH y 1Password Secret** nos dirigimos a **Dashboard>Manage Jenkins>Plugins** damos click en _Available plugins_.

Seguimos con el siguiente paso que va ser el de crear las credenciales de los servidores: _App Server 3 y Storage Server_. Para ello debemos estar ubicamdos en **Dashboard>Manage Jenkins>Credentials** y damos click sobre **global** luego damos click sobre el boton **Add Credentials**. En _Kind_ lo dejamos en **Username with password**, luego rellenamos el formulario con el usuario y contrasena del servidor App 3 y por ultimo damos click en el boton **Create**. Del mismo modo para el servidor de Almacenamiento.

Despues de todo esto vamos a crear **Secret text** para ambos servidores que en este caso van a ser las contrasenas. En _Kind_ vamos a seleccionar **Secret text**, en **Secret** debemos introducir nuestro secreto que va ser la contrasena del servidor, luego damos click en el boton **Create**.

Ahora nos dirigimos a **Dashboard>Manage Jenkins>System** en el apartado _SSH remote hots_ damos click en **Add**. En _Hostname_ colocamos la **IP** del servidor App 1 y en **Port** ingresamos 22 y en Credentials selecionamos el credencial previo que creamos en este caso es _banner (servidor apache)_ damos click sobre el boton **Save**.
Lo mismo vamos hacer para el servidor de almacenamiento.

Seguimos con el siguiente paso que va ser ir a **Dashboard** y damos click sobre _copy-logs_ y luego damos click sobre **Configure**, luego nos dirigimos al apartado **Triggers** y marcamos _Build periodically_ y colocamos:
```
H/5 * * * *
```

Luego nos vamos al apartado **Environment** y marcamos _Use secret text(s) or file(s)_, para ambos servidores: App 3 y Storage Server, damos click sobre **Add** y selecionamos **Secret text**, en _Variable_ le vamos a colocar PASS_APP y en _Credentials_ le vamos asignar **servidor apache**. Y para el otro servidor hacemos lo mismo solo que cambiamos _Variable_ con PASS_STORAGE y _Credentials_ con **servidor de almacenamiento**.

En el apartado de **Environment** marcamos tambien _Execute shell script on remote host using ssh_ y en **SSH site** seleccionamos _natasha@IP_. En _Pre build script_ colocamos:
```
echo $PASS_APP | sudo -S yum install sshpass
ls /var/log/httpd/
sshpass -p $PASS_STORAGE scp -o StrictHostKeyChecking=no /var/log/httpd/access_log natasha@172.16.238.15:/usr/src/finance/
sshpass -p $PASS_STORAGE scp -o StrictHostKeyChecking=no /var/log/httpd/error_log natasha@172.16.238.15:/usr/src/finance/
```



Ahora nos dirigimos al apartado **Build Steps**, damos click sobre _Add build step_ y seleccionamos **Execute shell script on remote host using ssh**, en _SSH site_ seleccionamos natasha@IP y colocamos los comandos:
```
ls -la /usr/src/finance
```

Finalmente damos click sobre el boton **Save**.



echo $PASS_APP | sudo -S yum install sshpass -y
ls /var/log/httpd/
sshpass -p $PASS_STORAGE scp -o StrictHostKeyChecking=no /var/log/httpd/access_log natasha@172.16.238.15:/usr/src/finance/


echo $PASS_STORAGE | sudo -S chmod -R 777 /usr/src/finance

ls -la /usr/src/finance