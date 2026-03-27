Vamos a diagnosticar si podemos llegar desde **jump_host**
```
telnet stapp01 6100
```

En esta caso no tenemos llegada al servidor **stapp01**.

## Verificar si tenemos apache2 o httpd

Inspeccionamos si tiene instalado apache2 o httpd, en este caso se tiene instalado httpd.
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