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
