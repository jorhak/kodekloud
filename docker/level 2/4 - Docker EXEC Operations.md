```
One of the Nautilus DevOps team members was working to configure services on a `kkloud` container that is running on `App Server 2` in `Stratos Datacenter`. Due to some personal work he is on PTO for the rest of the week, but we need to finish his pending work ASAP. Please complete the remaining work as per details given below:

  

a. Install `apache2` in `kkloud` container using `apt` that is running on `App Server 2` in `Stratos Datacenter`.  
  

b. Configure Apache to listen on port `8082` instead of default `http` port. Do not bind it to listen on specific IP or hostname only, i.e it should listen on localhost, 127.0.0.1, container ip, etc.  
  

c. Make sure Apache service is up and running inside the container. Keep the container in running state at the end.
```
# 1. Ingresar al servidor
```bash
ssh banner@stapp03
```
# 2. Listar contenedores
```bash
docker ps
```
# 3. Ingresar al contenedor
```bash
docker exec -it kkloud bash
```
#### 3.1 Actualizamos repositorio y instalamos HTTP
```bash
apt update && apt install apache2 vim -y
```
#### 3.2 Configurar puerto
```bash
vi /etc/apache2/ports.conf
```
#### Antes
```
Listen 80
```
#### Despues
```
Listen 3003
```
#### 3.3 Configurar hostname
```bash
vi /etc/apache2/sites-enabled/000-default.conf
```
#### Antes
```
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
```
#### Despues
```
        ServerName 127.0.0.1
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
```
#### 3.3 Configurar hostname global
```bash
vi /etc/apache2/apache2.conf
```
#### Antes
```
# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
#### Despues
```
# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
ServerName 127.0.0.1
```
### 3.4 Iniciar servicio
```bash
apache2ctl -k start
```
# 4. Verificar
```bash
curl localhost:3003
```