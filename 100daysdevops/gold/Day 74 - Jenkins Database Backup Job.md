```
There is a requirement to create a Jenkins job to automate the database backup. Below you can find more details to accomplish this task:



Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and password Adm!n321.


Create a Jenkins job named database-backup.


Configure it to take a database dump of the kodekloud_db01 database present on the Database server in Stratos Datacenter, the database user is kodekloud_roy and password is asdfgdsd.


The dump should be named in db_$(date +%F).sql format, where date +%F is the current date.

Copy the db_$(date +%F).sql dump to the Backup Server under location /home/clint/db_backups.


Further, schedule this job to run periodically at */10 * * * * (please use this exact schedule format).


Note:


You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case please make sure to refresh the UI page.


Please make sure to define you cron expression like this */10 * * * * (this is just an example to run job every 10 minutes).


For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

Luego de ingresar a **Jenkins** lo que hacemos es dar click sobre **Create a job**, despues ingresamos el nombre propuesto y seleccionamos **Freestyle project** y presionamos **OK**.

Seguido de eso vamos a actulizar los _plugins_ y a instalar los plugins **SSH y 1Password Secrets**.
Por lo cual debemos estar en **Dashboard>Manage Jenkins>Plugins** y en _Updates_ actualizamos los plugins. Luego vamos a instalar los plugins ya mencionados y para ello nos vamos a _Available plugins_ y buscamos en la barra de busqueda.

El siguiente paso sera crear _secret text_ de: Database server, Backup server y MariaDB password.
Por lo cual nos dirigimos a **Dashboard>Manage Jenkins>Credentials** damos click sobre _global_, seguido damos click en **Add Credentials** y vamos seleccionar en _Kind_:Secret text y rellenamos el formulario. Los secretos van a ser las contrasenas de los usuarios y la contrasena para acceder a la base de datos. 

Ahora vamos a configurar el **job**, debemos estar ubicados en **Dashboard** y dar click sobre _database-backup_ y luego damos click en **Configure**.
Nos dirigimos al apartado de _Triggers_ y marcamos _Build periodically_ en donde debemos ingresar:
```
*/10 * * * *
```

Vamos a utilizar los secrets text como variables nos dirigimos al apartado de **Environment** y marcamos _Use secret text(s) or file(s)_ luego damos click en **Add** y seleccionamos _Secret text_ y rellenamos el formulario tanto para **db (PASS_DB), backup (PASS_BACKUP) y mariadb (PASS_MARIADB)**.

Luego nos vamos al apartado **Build Steps** y damos click sobre _Add build step_ y seleccionamos _Execute shell_, lo primero que vamos hacer es crear una variable para utilizarla como nombre del backup de nuestra DB, luego vamos a ejecutar el comando que se va ejecutar en el servidor **DB** que nos va generar el backup, finalmente vamos a copiarlo en el servidor **Backup**. Para no tener el fichero .sql en nuestro _WORKSPACE_ lo vamos a eliminar ya que lo vamos a copiar en otro servidor, el ultimo comando verifica que se copio el fichero en el servidor **Backup**:
```
DB_FILE="db_$(date +%F).sql"

sshpass -p "$PASS_DB" ssh -o StrictHostKeyChecking=no peter@172.16.239.10 "mariadb-dump -u kodekloud_roy --password='$PASS_MARIADB' kodekloud_db01" > "$DB_FILE"
ls -la

sshpass -p "$PASS_BACKUP" scp -o StrictHostKeyChecking=no "$DB_FILE" clint@172.16.238.16:/home/clint/db_backups

rm "$DB_FILE"

sshpass -p "$PASS_BACKUP" ssh -o StrictHostKeyChecking=no clint@172.16.238.16 "ls -la /home/clint/db_backups"
```

Finalmente damos click en **Save**
Verificamos la ejecucion presionando **Build Now**
