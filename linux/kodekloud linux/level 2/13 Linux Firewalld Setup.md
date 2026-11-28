```
To secure our `Nautilus` infrastructure in `Stratos Datacenter`, we have decided to install and configure `firewalld` on one of the app servers named App Server `2`. We have Apache and Nginx services running on these apps. Nginx is running as a reverse proxy server for Apache. We might have more robust firewall settings in the future, but for now we have decided to go with the given requirements listed below:  
  

  

a. Allow all incoming connections on Nginx port, i.e `80`.  
  
b. Block all incoming connections on Apache port, i.e `8085`.  
  
c. All rules must be permanent.  
  
d. Zone should be public.  
  
e. If Apache or Nginx services aren't running already, please make sure to start them.
```
# Ingresar a servidor
```bash
ssh steve@stapp02
```
# Instalar firewalld
```bash
sudo yum install -y firewalld
```
## Habilitar y inicializar servicio
```bash
sudo systemctl enable firewalld
sudo systemctl start firewalld
sudo systemctl status firewalld
```
# Verificar servicios
```bash
sudo systemctl status httpd
sudo systemctl status nginx
```

```bash
cat /etc/httpd/conf/httpd.conf
cat /etc/nginx/nginx.conf
```
Podemos ver que ambos servicios estan usando el mismo puerto y es por eso que el servicio de Nginx no esta levantado, vamos a modificar la configuracion de **httpd** para que se ejecute por el puerto **8085**.
```bash
sudo vi /etc/httpd/conf/httpd.conf
```
#### Before
```text
Listen 80
```
#### After
```text
Listen 8085
```
#### Reiniciar y ver estado de los servicios
```bash
sudo systemctl restart httpd
sudo systemctl restart nginx
sudo systemctl status httpd
sudo systemctl status nginx
```
# Configurar firewalld
```bash
sudo firewall-cmd --list-all-zones
sudo firewall-cmd --zone=public --permanent --add-port=80/tcp
sudo firewall-cmd --zone=public --permanent --remove-port=8085/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --list-all
```