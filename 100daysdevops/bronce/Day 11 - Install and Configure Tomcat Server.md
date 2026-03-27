Nos solicitan desplegar una app que fue hecha en Java. Para ello vamos a utilizar Tomat Server.
Lo que vamos hacer es ver si tenemos disponible Tomcat desde el repositorio de paquetes.
```
sudo dnf search tomcat
```

Instalar Tomcat
```
sudo dnf install tomcat -y
```


Ahora debemos habilitar y inicar el servicio
```
sudo systemctl enable tomcat
sudo systemctl start tomcat
```

Verificar el estado del servicio
```
sudo systemctl status tomcat
```

## Modificar puerto

Primero debemos detener el servico
```
sudo systemctl stop tomcat
```

Modificar fichero
```
sudo vi /etc/tomcat/server.xml
```

Tiene que quedar de esta forma con el puerto 3001
```
<Connector port="3001" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
               />
```

Levantar el servico
```
sudo systemctl start tomcat
```
## Copiar ROOT.war
Dado que tenemos esta fichero en otro server debemos copiarlo a nuestro servidor. Abrimos otra terminal y ingresamos en el servidor que contiene a ROOT.war
```
ssh thor@jump_host
cd /tmp
scp ROOT.war steve@172.16.238.11:/home/steve/
```

Ahora volvemos a la otra terminal y verificamos que ya tenemos el fichero
```
cd /home/steve
ls -la
```

## Desplegar ROOT.war

Antes debemos detener el servicio

```
sudo systemctl stop tomcat
```

Lo que debemos hacer es mover nuestro .war a la ruta /usr/share/tomcat/webapps

```
sudo mv ROOT.war /usr/share/tomcat/webapps/
```

Ahora levantamos el servico
```
sudo systemctl start tomcat
```

## Probar App
```
curl http://stapp02
```

Debemo ver esto
```
<!DOCTYPE html>
<!--
To change this license header, choose License Headers in Project Properties.
To change this template file, choose Tools | Templates
and open the template in the editor.
-->
<html>
    <head>
        <title>SampleWebApp</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <h2>Welcome to xFusionCorp Industries!</h2>
        <br>
    
    </body>
</html>
```
