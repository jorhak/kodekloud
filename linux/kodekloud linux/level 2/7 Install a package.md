```
As per new application requirements shared by the `Nautilus` project development team, serveral new packages need to be installed on all app servers in `Stratos Datacenter`. Most of them are completed except for `chrony`.  
  

  

Therefore, install the `chrony` package on all `app-servers`.
```
# Ingresar al servidor
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```
# Instalar paquete
```bash
sudo yum install -y chrony
```
# Verificar
```bash
sudo systemctl status chronyd
```