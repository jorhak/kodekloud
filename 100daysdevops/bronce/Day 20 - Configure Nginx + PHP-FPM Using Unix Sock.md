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