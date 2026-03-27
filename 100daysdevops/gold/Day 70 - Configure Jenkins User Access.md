```
The Nautilus team is integrating Jenkins into their CI/CD pipelines. After setting up a new Jenkins server, they're now configuring user access for the development team, Follow these steps:  
  
1. Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login with username `admin` and password `Adm!n321`.

2. Create a jenkins user named `kareem` with the password`LQfKeWWxWD`. Their full name should match `Kareem`.  
  
3. Utilize the `Project-based Matrix Authorization Strategy` to assign `overall read` permission to the `kareem` user.  
  
4. Remove all permissions for `Anonymous` users (if any) ensuring that the `admin` user retains overall `Administer` permissions.  
  
5. For the existing job, grant `kareem` user only `read` permissions, disregarding other permissions such as Agent, SCM etc.  
  
`Note:`  
  
6. You may need to install plugins and restart Jenkins service. After plugins installation, select `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page.  

  
7. After restarting the Jenkins service, wait for the Jenkins login page to reappear before proceeding. Avoid clicking `Finish` immediately after restarting the service.  
  
  
8. Capture screenshots of your configuration for review purposes. Consider using screen recording software like `loom.com` for documentation and sharing.
```

# Crear usuario
Despues de haber ingresado damos click en **Manage Jenkins**>**Users**>**Create User**, luego rellenamos el formulario con los datos provistos. Y pulsamos el boton **Create User**.

Luego actualizamos los _pluigns_ que estan desactualizados y reiniciamos **Jenkins**. 

Ahora debemos instalar el _plugin_ necesario para los permisos. Estamos en **Dashboard**, damos click en **Manage Jenkins**>**Plugins**>**Available plugins** y en la barra de busqueda escribimos **Matrix Authorization**, marcamos esta opcion y damos click en el boton **Install**. Al igual que en la actualizacion de los _plugins_ vamos a marcar el check de reinicio una ves se termine la instalacion del _plugin_.

Con el plugin instalado procedemos con los siguientes pasos, vamos a configurar al usuario **kareen** con los permisos que son requeridos. Para ellos nos ubicamos en **Dashboard**, damos click en **Manage Jenkins**>**Security**, buscamos el apartado **Authorization** y desplegamos la opciones y seleccionamos **Project-based Matrix Authorization Strategy**, agregamos al usuario _kareem_, damos click en el boton **Add user...** y luego introducimos el **User ID** que seria _kareem_ y damos click en el boton **OK**. Finalmente damos click en **Save**.

Siguiendo con el siguiente paso vamos a configurar el proyecto para que el usuario **siva** solo tenga permiso: **READ**.
Nos ubicamos en **Dashboard** luego damos click en **Helloworld**>**Configure** marcamos **Enable project-based security**, al igual que en el paso previo vamos a agregar a nuestro usuario **kareem**. Y marcamos en el area **Job**>**Read**
