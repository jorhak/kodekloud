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

