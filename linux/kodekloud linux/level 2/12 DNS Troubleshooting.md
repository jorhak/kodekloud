```
The system admins team of xFusionCorp Industries has noticed intermittent issues with DNS resolution in several apps . `App Server 2` in `Stratos Datacenter` is having some DNS resolution issues, so we want to add some additional DNS nameservers on this server.  
  

  

As a temporary fix we have decided to go with Google public DNS (ipv4). Please make appropriate changes on this server.
```
# Ingresar al servidor
```bash
ssh steve@stapp02
```
# Configurar resolv.conf
```bash
sudo vi /etc/resolv.conf
```

```text
nameserver 8.8.8.8
```
# Verificar
```bash
sudo yum update
sudo yum install -y bind-utils
```

```bash
nslookup google.com
```