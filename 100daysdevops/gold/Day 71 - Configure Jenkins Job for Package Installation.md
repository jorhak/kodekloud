```
Some new requirements have come up to install and configure some packages on the Nautilus infrastructure under Stratos Datacenter. The Nautilus DevOps team installed and configured a new Jenkins server so they wanted to create a Jenkins job to automate this task. Find below more details and complete the task accordingly:  
  

  

1. Access the Jenkins UI by clicking on the `Jenkins` button in the top bar. Log in using the credentials: username `admin` and password `Adm!n321`.  
  

2. Create a new Jenkins job named `install-packages` and configure it with the following specifications:  
  

- Add a string parameter named `PACKAGE`.
- Configure the job to install a package specified in the `$PACKAGE` parameter on the `storage server` within the `Stratos Datacenter`.  
      
    

`Note`:  
  

1. Ensure to install any required plugins and restart the Jenkins service if necessary. Opt for `Restart Jenkins when installation is complete and no jobs are running` on the plugin installation/update page. Refresh the UI page if needed after restarting the service.  
  

2. Verify that the Jenkins job runs successfully on repeated executions to ensure reliability.  
  

3. Capture screenshots of your configuration for documentation and review purposes. Alternatively, use screen recording software like `loom.com` for comprehensive documentation and sharing.
```

Luego de ingresar a **Jenkins** con la credenciales proporcionadas damos click en **Create a job** y colocamos el nombre que nos asignaron _install-packages_ y seleccionamos **Freestyle project**, luego damos click en el boton **OK**.

Actualizamos los _plugins_ que necesitan ser actualizados y reiniciamos **Jenkins**.
Instalamos los _plugins_ **SSH y 1Password Secrets** y reiniciamos **Jenkins**.

Debemos agregar los credenciales del servidor donde vamos a ejecutar los comandos. Nos encontramos en **Dashboard** damos click en **Manage Jenkins** nos dirigimos en el apartado de **Security** y damos click en **Credentials**>**global**, luego damos click en el boton **Add Credentials**.
En _Kind_ lo dejamos en **Username with password** agregamos el usuario que tenemos para ingresar **storage server** rellenamos el formulario y damos click en el boton **Create**.

Vamos a realizar lo mismo solo que vamos a cambiar _Kind_: Secret text, y rellemanos el formulario, el secreto debe ser la contrasena del usuario en donde vamos a ejecutar los comandos.

Agregamos el servidor en donde vamos a ejecutar el comando. Estamos en **Dashboard**>**Manage Jenkins**>**System** nos  dirigimos al apartado _SSH remote hosts_ y damos click en **Add**, en **hostname** agregamos la **IP**: 172.16.238.15 el **puerto**: 22 y los credenciales que va usar en nuestro caso es **natasha (storage server)**, y finalmente damos click en **Save**.
De igual modo nos vamos al partado de _1Password secret_ y agregamos la **IP**.

Nos ubicamos en **Dashboard** damos click en **install-package**>**Configure** marcamos **This project is parameterized**>**Add Parameter**>**String Parameter** y rellenamos el formulario con los parametros asignados.

Luego nos dirigimos a **Build Steps** damos click en **Add build step**>**Execute shell script on remote host using ssh**

Despues nos dirigimos al apartado de **Environment** y marcamos _Use secret text(s) or file(s)_ rellenamos el formulario, de igual modo marcamos _Execute shell script on remote host using ssh_ y rellenamos el formulario.
!image[image](hola)
