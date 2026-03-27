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