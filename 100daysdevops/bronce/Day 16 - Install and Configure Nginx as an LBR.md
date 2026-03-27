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
