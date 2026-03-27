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
