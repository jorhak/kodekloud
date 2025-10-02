Entramos al reto de 100 dias de DevOps, aqui se van a tener los primeros 25 dias del reto.

# Day 1: Linux User Setup with Non-Interactive Shell
```
sudo useradd -s /sbin/nologin mark
```

# Day 2: Temporary User Setup with Expiry
```
sudo useradd -e 2024-04-15 -c "Yousuf Ari" -d /home/yousuf -s /bin/bash yousuf
```

# Day 3: Secure Root SSH Access
```
sudo vi /etc/ssh/sshd_config
```

Before
```
# Authentication:

#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

After
```
# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

Reiniciar servicio
```
sudo systemctl restart sshd
```

# Day 4: Script Execution Permissions
```
sudo chmod 755 xfusioncorp.sh
```

# Day 5: SElinux Installation and Configuration
```
sudo yum update
sudo yum upgrade

sudo yum install policycoreutils selinux-policy setroubleshoot -y

#Alternativa
#sudo yum install policycoreutils selinux-policy selinux-policy-targeted 
```

## Configurar SELinux mode
Lo que vamos a hacer sera habilitar el modo permisivo o forzado:
- Permisivo: registra las acciones pero no las impone
- Forzado: registra las acciones y las bloquea
```
sudo nano /etc/selinux/config
SELINUX=permissive
```


# Day 6: Create a Cron Job
```
sudo yum update && sudo yum install cronie -y

sudo crontab -e
```

## Agregar
```
*/5 * * * * echo hello > /tmp/cron_text
```

## Verificar
```
sudo crontab -l
```

# Day 7: Linux SSH Authentication
Primero debemos ubicarnos en el servidor del cual nos vamos a conectar a los otros servidores en nuestro caso es el servidor **jump_host**

```
ssh thor@jump_host
```

## Configurar autenticacion sin contrasena
```
sudo visudo
```

### Agregar una linea similar reemplazando sudo_user por el usuario thor
```
thor ALL=(ALL) NOPASSWD: ALL
```

## Generar llaves
```
ssh-keygen -t rsa -b 4096
```

### Copiar la llave publica en todos los servidores
```
ssh-copy-id -i ~/.ssh/id_rsa.pub tony@172.16.238.10
ssh-copy-id -i ~/.ssh/id_rsa.pub steve@172.16.238.11
ssh-copy-id -i ~/.ssh/id_rsa.pub banner@172.16.238.12
```

## Probar conexion
```
ssh tony@172.16.238.10
ssh steve@172.16.238.11
ssh banner@172.16.238.12
```


# Day 8: Install Ansible
## Instalar pipx
```
sudo dnf install pipx 
pipx ensurepath 
```

```
pipx install --include-deps ansible

pipx install ansible-core==4.7.0
```

Con los comandos anteriores no funciono ya que no tiene esa version en especifico, solo se tiene hasta la 2.15.13.

## Alternativa
```
sudo yum update
sudo yum install python3 python3-pip -y
```

## Instalar ansible
```
sudo pip3 install ansible==4.7.0
```

### Verificar instalacion
```
ansible --version
```

Cometi un error con al anterior instalacion con pipx 2.15.13 es equivalenta a 4.7.0 esta es la verison del comprimido y ya compilado tiene la version 2.15.13.
Lo cual es lo mismo crei que con pipx tenia versiones viejas.

# Day 9: MariaDB Troubleshooting
Aqui tenemos un error ya que MariaDB no esta activo. Esta instalado pero al parecer tiene un error y lo debemos analizar para poder resolverlo.

Se empeso a revisar los logs
```
sudo journalctl -u mariadb.service --since "15 minutes ago"
```

En donde se encontro que no teniamos permisos 
```
ep 10 19:41:14 stdb01.stratos.xfusioncorp.com mariadbd[2712]: 2025-09-10 19:41:14 0 [Warning] Can't create test file '/var/lib/mysql/stdb01.lower-test' (Errcode: 13 "Permission denied")
```

El cual resolvimos con
```
sudo chown -R mysql:mysql /var/lib/mysql
```

Verificar
```
sudo systemctl restart mariadb
sudo systemctl status mariadb
```

# Day 10: Linux Bash Scripts

### Permisos
Lo primero que hicimos fue configurar la autenticacion sin contrasena ya que el script necesitaba que se ejecuten comandos sin contrasena.
Luego seguimos con la generacion de llaves SSH para enviar el fichero .zip al servidor backup.
Todo esto lo hicimos con la documentacion del dia 7.

Instalar zip
```
sudo dnf install zip -y
```

```
#!/bin/bash
#Script para realizar un backup del directorio
#/var/www/html/news

zip -r xfusioncorp_news.zip /var/www/html/news
mv xfusioncorp_news.zip /backup/
cd /backup
scp xfusioncorp_news.zip clint@172.16.238.16:/backup/
```

## Dar permisos
```
chmod 755 news_backup.sh
```

## Ejecutar script
```
./news_backup.sh
```

## Validar
Abrimos otra terminal y accedemos por SSH a servidor backup y nos dirigimos a la ruta _/backup_ en donde veremos nuestro comprimido _xfusioncorp_news.zip_.

# Day 11: Install and Configure Tomcat Server

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


# Day 12: Linux Network Services

Vamos a diagnosticar si podemos llegar desde **jump_host**
```
telnet stapp01 6100
```

En esta caso no tenemos llegada al servidor **stapp01**.

## Verificar si tenemos apache2 o httpd

Inspeccionamos si teninia instalado apache2 o httpd, en este caso se tiene instalado httpd.
Al momento de ver el estado del servicio nos muestra que 
- No puede ser vinculado a la direccion 0.0.0.0:8082
- No esta escuchando el sockets habilitado.

```
sudo systemctl status httpd
```

```
Sep 15 23:59:55 stapp01.stratos.xfusioncorp.com httpd[510]: (98)Address alre
ady in use: AH00072: make_sock: could not bind to address 0.0.0.0:8082
Sep 15 23:59:55 stapp01.stratos.xfusioncorp.com httpd[510]: no listening soc
kets available, shutting down
Sep 15 23:59:55 stapp01.stratos.xfusioncorp.com httpd[510]: AH00015: Unable 
to open logs
Sep 15 23:59:55 stapp01.stratos.xfusioncorp.com systemd[1]: 
httpd.service: Child 510 belongs to httpd.service.
Sep 15 23:59:55 stapp01.stratos.xfusioncorp.com systemd[1]: 
httpd.service: Main process exited, code=exited, status=1/FAILURE
Sep 15 23:59:55 stapp01.stratos.xfusioncorp.com systemd[1]: 
httpd.service: Failed with result 'exit-code'.
Sep 15 23:59:55 stapp01.stratos.xfusioncorp.com systemd[1]: 
httpd.service: Changed start -> failed
```

## Habilitar puertos
Nos encontramos con un problema y es que no tenemos instalado _firewalld_ y no lo podemos instalar.
```
sudo firewall-cmd --permanent --add-port=6100/tcp
sudo firewall-cmd --reload
```

## Verificar SELinux
```
sudo getenforce
sudo semanage port -a -t http_port_t -p tcp 6100
```

Al igual que en el _firewalld_ no lo tengo instalado por lo que no lo puedo utilizar ni instalar.
## Verificar que otro servicio no este utilizando el puerto
```
sudo netstat -tulpn | grep 6100
```

Encontramos un servico que esta utiliznado el puerto. Lo que debemos hacer es detener el servicio.
```
sudo systemctl stop sendmail
```

## Inicar servicio httpd
```
sudo systemctl start httpd
```

## Probar desde jump_host
```
telnet stapp01 6100
curl http://stpapp01:6100
```

Podemos ver que seguimos sin poder conectarnos.

## Alternativa
Ya que no podemos utilizar _firewalld y semanage_ vamos a intentar con **iptables**.

### Listar las reglas
```
sudo iptables -L -n -v
```

### Anadir una regla para iptables
```
sudo iptables -A INPUT -p tcp --dport 6100 -j ACCEPT
```

### Guardar las reglas de iptables
```
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

Volvemos a probar y seguimos con el mismo problema no tenemos conexion.

# Anadir regla al inicio
```
sudo iptables -I INPUT -p tcp --dport 6100 -j ACCEPT
```

## Verificar regla
```
sudo iptables -L INPUT -n -v
```

Volvemos a probar y ahora si tenemos conexion.
El error era que la primer regla que utilizamos la agregaba con -A (append) que podria estar siendo ignorada porque una regla anterior esta bloqueando el trafico.
La opcion -I (insert), se asegura que la regla se coloque al inicio de la cadena donde tiene prioridad sobre cualquier regla de _drop o reject_ que se tenga.

# Day 13: IPtables Installation And Configuration

## Verificar puerto expuesto
```
curl 172.16.238.10:8086
curl 172.16.238.11:8086
curl 172.16.238.12:8086
curl 172.16.238.14:8086
```

En la ultima **IP** tubimos un incoveniente no nos pudimos conectar, vamos a solucionarlo mas adelante.

## Instalar IPTABLES
### Acceder a los servidores por ssh
```
ssh tony@172.16.238.10
```

## Verificar si esta instalado IPTABLES
```
iptables --version
```

Ya tiene instalada una version que es un legado asi que vamos a instalar una nueva.

## Instalacion de iptables (por si no tienen instalado)
```
sudo dnf install iptables -y
```

## Configurar IPTABLES
```
sudo iptables -I INPUT -p tcp --dport 8086 -j REJECT
sudo iptables -I INPUT -p tcp --dport 8086 -s 172.16.238.14 -j ACCEPT
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

## Asegurar IPTABLES
```
sudo dnf install iptables-services -y
sudo systemctl enable iptables
sudo systemctl start iptables
sudo service iptables save
```

Repetimos todo desde **Instalar IPTABLES** en los otros servidores.
## Para el LBR
### Probar llegada
```
ssh loki@172.16.238.14
```

```
curl 172.16.238.10:8086
curl 172.16.238.11:8086
curl 172.16.238.12:8086
```

# Day 14: Linux Process Troubleshooting
## Verificar estado de APACHE2 en los servidores
```
curl -I 172.16.238.10:8089
curl -I 172.16.238.11:8089
curl -I 172.16.238.12:8089
```

Podemos observar que el servidor **172.16.238.10** es el que nos muestra que el servicio de _apache2_ no esta respondiendo.

## Ingresar al servidor
```
ssh tony@172.16.238.10
```

## Ver estado del servicio
```
sudo systemctl status httpd
```

## Verificar si otro servicio no esta utilizando el puerto
```
sudo netstat -ntlp | grep 8089
```

## Detener servicio que esta utilizando el puerto
```
sudo systemctl stop sendmail
```

## Reiniciar servicio HTTPD
```
sudo systemctl restart httpd
```

# Day 15: Setup SSL for Nginx
## Ingresar al servidor
```
ssh steve@172.16.238.11
```
# Instalar nginx
```
sudo dnf install nginx -y
```
## Ver las configuraciones
```
sudo vi /etc/nginx/nginx.conf
```

Descomentamos todo lo relacionado a TLS
```
#Settings for a TLS enabled server.

    server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/pki/nginx/nautilus.crt";
        ssl_certificate_key "/etc/pki/nginx/private/nautilus.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

## Crear directorios
```
sudo mkdir /etc/pki/nginx
sudo mkdir /etc/pki/nginx/private
```
## Mover los certificados
```
sudo cp /tmp/nautilus.crt /etc/pki/nginx
sudo cp /tmp/nautilus.key /etc/pki/nginx/private
```

## Reiniciar servicio
```
sudo systemctl restart nginx
sudo systemctl status nginx
```

## Modificar index.html
```
sudo vi /usr/share/nginx/html/index.html
```

Seleccionar con vi **Esc > V** y con las flechas seleccionas hacia arriba o abajo. Y personasmos **Supr** para borrar el contenido seleccionado. Una vez seleccionado podemos borrar presionando **d** y podemos copiar presionando **y**. Para pegar persiona p (despues del cursor) o P (antes del cursor).

```
Welcome!
```

## Verificar
```
curl -Ik https://172.16.238.11
curl -k https://172.16.238.11
```
# Day 16: Install and Configure Nginx as an LBR
## Acceder al servidor
```
ssh loki@172.16.238.14
```

## Instalar Nginx
```
sudo dnf install nginx -y
```

## Verificar que Apache2 esta habilitado en los servidores
```
ssh tony@172.16.238.10
```

### Estatus de Apache2
```
sudo systemctl status httpd
```

### Verificar que puerto estan utilizando
```
cat /etc/httpd/conf/httpd.conf | grep Listen
```

```
curl 172.16.238.10:8087
curl 172.16.238.11:8087
curl 172.16.238.12:8087
```

Repetimos esto para todos los servidores.

## Configurar Nginx LBR
```
sudo vi /etc/nginx/nginx.conf
```

```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
	access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
    upstream backend_servers {
        # Define los servidores backend y el método de balanceo (round robin por defecto)
        server 172.16.238.10:8087;
        server 172.16.238.11:8087;
        server 172.16.238.12:8087;
    }

    server {
        listen       80;
        listen       [::]:80;
        server_name  172.16.238.14;
        location / {
            # Reenvía las peticiones al grupo upstream 'backend_servers'
            proxy_pass http://backend_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```

### Verificar que la configuracion es correcta
```
sudo nginx -t
```

### Reiniciar servicio
```
sudo systemctl restart nginx
```

# Verificar configuracion
```
curl 172.16.238.14:80
```

Ingresamos a los servidores y vemos a que server se envia la peticion
```
sudo tail -f /var/log/httpd/access_log
```

De igual manera en el servidor Nginx LBR
```
sudo tail -f /var/log/nginx/access.log
```

# Day 17: Install and Configure PostgreSQL
## Ingresar al servidor
```
ssh peter@172.16.239.10
```

## Verificar version de PostgreSQL
```
psql --version
```

## Conexion a la base de datos
```
psql -U postgres
```

## Crear usuario
```
CREATE USER kodekloud_joy WITH CREATEDB CREATEROLE LOGIN PASSWORD 'LQfKeWWxWD' ;
```

## Asignar todos los permisos
```
ALTER USER kodekloud_joy SUPERUSER;
```

## Ver privilegios asignados al usuario
```
/du
```

## Ingresar con usuario
```
psql -U kodekloud_joy -d postgres
```

## Crear base de datos
```
CREATE DATABASE kodekloud_db7;
```

### Asignar usuario a la base de datos
```
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_joy;
```

### Ver los permisos de la base de datos
```
\l
```

# Day 18: Configure LAMP server
## Ingresamos al servidor
```
ssh tony@172.16.238.10
```

## Verificamos fichero
```
cat /var/www/html/index.php
```

## Instalar HTTPD y PHP
```
sudo dnf install httpd php php-mbstring php-mysqlnd php-pdo -y
```

## Cambiar directorio index (opcional)
Buscar el fichero.
```
sudo vi /etc/httpd/mods-enabled/dir.conf
```

```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
### Cambiar puerto
```
sudo vi /etc/httpd/conf/httpd.conf
```

```
Listen 6300
```

### Reiniciar y ver estado del servicio
```
sudo systemctl restart httpd
sudo systemctl status httpd
```

Repetir en los otros servidores
## Instalar MySQL
### Conectar con servidor
```
ssh peter@172.16.239.10
```

### Instalacion de MySQL
```
sudo dnf install mariadb-server mariadb -y
```

### Habilitar y iniciar MySQL
```
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

### Configurar seguridad
```
sudo mysql_secure_installation
```
### Ingresar a MySQL
```
mysql -u root -p
```
## Crear usuario
```
CREATE USER 'kodekloud_cap'@'%' IDENTIFIED BY 'Rc5C9EyvbU';
```

## Crear base de datos
```
CREATE DATABASE kodekloud_db4;
```

## Asignar permisos a la base de datos
```
GRANT ALL PRIVILEGES ON kodekloud_db4.* TO 'kodekloud_cap'@'%';
```

## Aplicar cambios
```
FLUSH PRIVILEGES;
```

## Ingresar a la base de datos
```
mysql -u kodekloud_cap -p
```

# Day 19: Install and Configure Web Application
## Ingresamos al servidor
```
ssh banner@172.16.238.12
```

## Instalar HTTPD
```
sudo dnf install httpd -y
```

### Configurar puerto
```
sudo vi /etc/httpd/conf/httpd.conf
```

```
Listen 5004
```

### Habilitar y reiniciar servicio
```
sudo systemctl enable httpd
sudo systemctl start httpd
```

## Copiar paginas desde jump_host
```
 scp -r apps/ banner@172.16.238.12:/home/banner
 scp -r apps/ banner@172.16.238.12:/home/banner
```

### Mover directorios a /var/www/html
#### Ingresar a servidor
```
ssh banner@172.16.238.12
```

#### Copiar directorios
```
sudo cp -r apps/ /var/www/html
sudo cp -r ecommerce /var/www/html
```

### Crear virtualhost
```
sudo vi /etc/httpd/conf/httpd.conf
```

```
<VirtualHost *:5004/apps>
    DocumentRoot "/var/www/html/apps"
    ServerName localhost
    CustomLog /var/log/httpd/apps_access.log combined
    ErrorLog /var/log/httpd/apps_error.log
</VirtualHost>

<VirtualHost *:5004/ecommerce>
    DocumentRoot "/var/www/html/ecommerce"
    ServerName localhost
    CustomLog /var/log/httpd/ecommerce_access.log combined
    ErrorLog /var/log/httpd/ecommerce_error.log
</VirtualHost>
```

#### Reiniciar HTTPD
```
sudo systemctl restart httpd
```

## Verificar
```
curl http://localhost:5004/apps/
curl http://localhost:5004/ecommerce/
```

# Day 20: Configure Nginx + PHP-FPM Using Unix Sock
## Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Instalar Nginx
```
sudo dnf install nginx -y
```

### Habilitar y arrancar Nginx
```
sudo systemctl enable nginx
sudo systemctl start nginx
```
### Configurar puerto
```
sudo vi /etc/nginx/nginx.conf
```

```
server {
        listen       8099;
        listen       [::]:8099;
        server_name  stapp03;
        root         /var/www/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

### Verificar sintaxis y reiniciar servicio
```
sudo nginx -t
sudo systemctl restart nginx
```
# Instalar php-fpm
Aqui nosotros tenemos la version que viene del repositorio de paquetes.
### Ver paquete del repositorio
```
sudo dnf search php-fpm
sudo dnf info php-fpm
```
Tenemos la version **8.0.30** cuando la especificacion dice que debe ser **8.1**.
Por eso debemos agregar el repositorio que tiene la version que se requiere.
Tenemos que saber que version de OS tenemos
```
cat /etc/os-release
```
Contamos con la version **9**: 
```
NAME="CentOS Stream"
VERSION="9"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="9"
PLATFORM_ID="platform:el9"
PRETTY_NAME="CentOS Stream 9"
ANSI_COLOR="0;31"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:centos:centos:9"
HOME_URL="https://centos.org/"
BUG_REPORT_URL="https://issues.redhat.com/"
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux 9"
REDHAT_SUPPORT_PRODUCT_VERSION="CentOS Stream"
```
Es por eso que vamos a utilizar **remi-release-9**.
## Instalar repositorio Remi (version 9)
```
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

## Instalar PHP-FPM 8.3
### Restablecer y habilitar el modulo PHP 8.3

```
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.3 -y
```

### Ver las versiones que tenemos disponibles
```
sudo dnf search php-fpm
```

```
Last metadata expiration check: 0:00:54 ago on Tue Sep 30 20:01:33 2025.
====================== Name Exactly Matched: php-fpm =======================
php-fpm.x86_64 : PHP FastCGI Process Manager
========================== Name Matched: php-fpm ===========================
php74-php-fpm.x86_64 : PHP FastCGI Process Manager
php80-php-fpm.x86_64 : PHP FastCGI Process Manager
php81-php-fpm.x86_64 : PHP FastCGI Process Manager
php82-php-fpm.x86_64 : PHP FastCGI Process Manager
php83-php-fpm.x86_64 : PHP FastCGI Process Manager
php84-php-fpm.x86_64 : PHP FastCGI Process Manager
php85-php-fpm.x86_64 : PHP FastCGI Process Manager
```
Ahora si podemos instalar la version que estan requiriendo.
### Instalar PHP-FPM y utilidades cli
```
sudo dnf install php81-php-fpm php81-php-cli -y
```

## Configurar php-fpm
#### Crear directorio
```
sudo mkdir -p /var/run/php-fpm/
sudo chown -R nginx:nginx /var/run/php-fpm
sudo chmod -R 755 /var/run/php-fpm
```

### Editar pool principal php-fpm
```
sudo vi /etc/opt/remi/php81/php-fpm.d/www.conf
```

```
listen = /var/run/php-fpm/default.sock
user = nginx	
group = nginx		
listen.owner = nginx
listen.group = nginx
listen.mode = 660	
```

#### Buscar directorio (por si acaso)
```
sudo find /etc -name "www.conf" 2>/dev/null
sudo find /var -type d -name php* 2>/dev/null
```
### Habilitar y arrancar php-fpm
```
sudo systemctl enable php81-php-fpm
sudo systemctl start php81-php-fpm
```

### Configurar Nginx y PHP-FPM para que trabajen juntos
```
sudo vi /etc/nginx/nginx.conf
```

```

location ~ \.php$ {
    # Requisito c: Usar el socket UNIX configurado en PHP-FPM
    fastcgi_pass unix:/var/run/php-fpm/default.sock; 

    # Archivos de inclusión FastCGI (asegúrate de que este archivo existe)
    include fastcgi_params;

    # Parámetro necesario para que PHP-FPM sepa qué archivo ejecutar
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_index index.php;
}
# Load configuration files for the default server block.
include /etc/nginx/default.d/*.conf;

error_page 404 /404.html;
location = /404.html {
}

error_page 500 502 503 504 /50x.html;
location = /50x.html {
}
```

#### Verificar sintaxis y reiniciar servicio
```
sudo nginx -t
sudo systemctl restart nginx
```

#### Reiniciar php-fpm
```
sudo systemctl restart php81-php-fpm
```

#### Asegurar permisos
```
sudo chown -R nginx:nginx /var/www/html
sudo chmod -R 755 /var/www/html
sudo chown -R nginx:nginx /var/opt/remi
sudo chmod -R 755 /var/opt/remi
```

## Prueba final
```
curl http://stapp03:8099/index.php
```

## Solucion rapida
```
sudo chmod 777 /var/run/php-fpm/default.sock
```

## Solucion recomendada

#### Ver el contenido del servicio
```
sudo systemctl cat php81-php-fpm
```

#### Editar servidio override
```
sudo systemctl edit php81-php-fpm
```

#### Anadir la configuracion del usuario
```
[Service]
User=nginx
Group=nginx
```

#### Aplicar cambios y probar
```
# 1. Recargar la configuración de systemd
sudo systemctl daemon-reload

# 2. Eliminar el socket antiguo para forzar la creación correcta
sudo rm -f /var/run/php-fpm/default.sock

# 3. Reiniciar el servicio PHP-FPM
sudo systemctl restart php81-php-fpm

# 4. Verificar la propiedad del socket
sudo ls -l /var/run/php-fpm/default.sock
```
Volvemos hacer la prueba final.
#### Lo que Ocurrirá (La Fusión de Configuraciones)
No vas a tener dos servicios ejecutándose ni dos archivos de servicio separados. systemd funciona mediante un mecanismo de anulación (override) y fusión.

- Archivo Original (Base): El archivo original instalado por el paquete Remi (/usr/lib/systemd/system/php83-php-fpm.service o similar) contiene la configuración básica, como el ExecStart que apunta al binario de PHP-FPM.

- Archivo de Anulación (Tu Corrección): El archivo que creaste en /etc/systemd/system/php83-php-fpm.service.d/override.conf contiene tu corrección (User=nginx, Group=nginx).

# Day 21: Set Up Git Repository on Storage Server
```
The Nautilus development team has provided requirements to the DevOps team for a new application development project, specifically requesting the establishment of a Git repository. Follow the instructions below to create the Git repository on the Storage server in the Stratos DC:



Utilize yum to install the git package on the Storage Server.


Create a bare repository named /opt/official.git (ensure exact name usage).
```
## Acceder al servidor
```
ssh natasha@172.16.238.15
```

## Instalar git
```
sudo dnf install git -y
```

## Crear repositorio
```
sudo git init --bare /opt/official.git
```

# Day 22: Clone Git Repository on Storage Server
```
The DevOps team established a new Git repository last week, which remains unused at present. However, the Nautilus application development team now requires a copy of this repository on the Storage Server in the Stratos DC. Follow the provided details to clone the repository:



The repository to be cloned is located at /opt/ecommerce.git


Clone this Git repository to the /usr/src/kodekloudrepos directory. Perform this task using the natasha user, and ensure that no modifications are made to the repository or existing directories, such as changing permissions or making unauthorized alterations.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Clonar repositorio
### Ir al directorio
```
cd /usr/src/kodekloudrepos
```

## Clonar repo

```
git clone /opt/ecommerce.git
```

# Day 23: Fork a Git Repository
```
There is a Git server utilized by the Nautilus project teams. Recently, a new developer named Jon joined the team and needs to begin working on a project. To begin, he must fork an existing Git repository. Follow the steps below:



1.-Click on the Gitea UI button located on the top bar to access the Gitea page.


2.-Login to Gitea server using username jon and password Jon_pass123.


3.-Once logged in, locate the Git repository named sarah/story-blog and fork it under the jon user.


Note: For tasks requiring web UI changes, screenshots are necessary for review purposes. Additionally, consider utilizing screen recording software such as loom.com to record and share your task completion process.
```

## Comenzar a grabar con loom

# Day 24: Git Create Branches
```
Nautilus developers are actively working on one of the project repositories, /usr/src/kodekloudrepos/blog. Recently, they decided to implement some new features in the application, and they want to maintain those new changes in a separate branch. Below are the requirements that have been shared with the DevOps team:



1.-On Storage server in Stratos DC create a new branch xfusioncorp_blog from master branch in /usr/src/kodekloudrepos/blog git repo.


2.-Please do not try to make any changes in the code.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

### Ir al repositorio
```
cd /usr/src/kodekloudrepos/blog
```

### Ver en que rama se encuentra
```
sudo git branch
```
Debe estar en la rama **master**.

### Cambiar de rama 
```
sudo git checkout master
```
### Crear rama
```
sudo git checkout -b xfusioncorp_blog
```


# Day 25: Git Merge Branches
```
The Nautilus application development team has been working on a project repository `/opt/ecommerce.git`. This repo is cloned at `/usr/src/kodekloudrepos` on `storage server` in `Stratos DC`. They recently shared the following requirements with DevOps team:  
  

  

Create a new branch `datacenter` in `/usr/src/kodekloudrepos/ecommerce` repo from `master` and copy the `/tmp/index.html` file (present on `storage server` itself) into the repo. Further, `add/commit` this file in the new branch and merge back that branch into `master` branch. Finally, push the changes to the origin for both of the branches.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Nos dirigimos al directorio donde este el repositorio
```
cd /usr/src/kodekloudrepos/ecommerce
```

## Verificar rama
```
sudo git branch
```
Debe ser la master.

## Cambiar de rama (sino esta en la rama master)
```
sudo git checkout master
```

## Crear rama
```
sudo git checkout -b datacenter
```

## Copiar fichero
```
sudo cp /tmp/index.html .
```

## Agregar los cambios al workspace y stage
```
sudo git add .
sudo git commit -m "Add index.html"
```

## Subir cambios de la rama datacenter
```
sudo git branch -M datacenter
sudo git push -u origin datacenter
```

## Realizar merge de datacenter a master
```
sudo git checkout master
sudo git merge datacenter
sudo git push
```