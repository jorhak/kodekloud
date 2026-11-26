```
We have some users on all app servers in `Stratos Datacenter`. Some of them have been assigned some new roles and responsibilities, therefore their users need to be upgraded with sudo access so that they can perform admin level tasks.  
  

  

a. Provide sudo access to user `mariyam` on all app servers.  
  

b. Make sure you have set up password-less sudo for the user.
```
# Ingresar a los servidores
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```
# Agregar a sudores
```bash
sudo visudo
```

```text
mariyam ALL=(ALL) NOPASSWD:ALL
```
# Verificar
```bash
sudo su - mariyam
sudo whoami
```