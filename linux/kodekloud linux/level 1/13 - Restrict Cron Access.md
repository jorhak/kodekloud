```
In alignment with security compliance standards, the Nautilus project team has opted to impose restrictions on crontab access. Specifically, only designated users will be permitted to create or update cron jobs.  
  

  

Configure crontab access on App Server 1 as follows: Allow crontab access to `mariyam` user while denying access to the `rod` user.
```

Ingresar al servidor:
```
ssh tony@172.16.238.10
```

Agregar usuario **mariyam**, para permitirle crear o actualizar _cron jobs_:
```
sudo vi /etc/cron.allow
```

Quitar usuario rod, para que no pueda crear o actualizar _cron jobs_:
```
sudo vi /etc/cron.deny
```

Intentar crear un crontab 
```
crontab -e
```
Nos va salir un error ya que el usuario con el que queremos crear un crontab no tiene permitido realizar esta accion.

Cambiar de usuario:
```
sudo su mariyam
```

Crear _crontab_:
```
crontab -e
```

```
* * * * * ls -la /home >> /tmp/ls.log
```

Nos cambiamos de directorio y verificamos que se ejecuta:
```
cd /tmp/
ls 
```