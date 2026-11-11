```
The system admins team of `xFusionCorp Industries` has set up some scripts on `jump host` that run on regular intervals and perform operations on all app servers in `Stratos Datacenter`. To make these scripts work properly we need to make sure the `thor` user on jump host has password-less SSH access to all app servers through their respective sudo users (i.e `tony` for app server 1). Based on the requirements, perform the following:  
  

  

Set up a password-less authentication from user `thor` on jump host to all app servers through their respective sudo users.
```
# Verificar usuario
Vamos a verificar que estamos utilizando el usuario **thor**.
```bash
whoami
```
# Crear llave SSH
```bash
ssh-keygen -t rsa -b 4096 -C "Thor Host"
```

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```
# Copiar llave a servidores
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub tony@stapp01
ssh-copy-id -i ~/.ssh/id_rsa.pub steve@stapp02
ssh-copy-id -i ~/.ssh/id_rsa.pub banner@stapp03
```
# Configurar servidores
Vamos a configurar los servidores para que ya no solicite contrasena sino que utilice el par de llaves. Se van a repetir estas configuraciones en los tres servidores.
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

```bash
sudo vi /etc/ssh/sshd_config
```
#### Descomentar
```bash
PermitRootLogin prohibit-password
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
PasswordAuthentication no
PermitEmptyPasswords no
```
#### Reiniciar servicio
```bash
sudo systemctl restart sshd
```