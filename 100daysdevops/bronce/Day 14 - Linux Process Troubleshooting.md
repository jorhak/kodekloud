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
