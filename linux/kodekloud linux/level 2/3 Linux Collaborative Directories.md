```
The Nautilus team doesn't want its data to be accessed by any of the other groups/teams due to security reasons and want their data to be strictly accessed by the `sysadmin` group of the team.  
  

  

Setup a collaborative directory `/sysadmin/data` on `app server 2` in `Stratos Datacenter`.  
  

The directory should be group owned by the group `sysadmin` and the group should own the files inside the directory. The directory should be `read/write/execute` to the user and group owners, and `others` should not have any access.
```
# Ingresar al servidor
```bash
ssh steve@stapp02
```
# Listar grupos
```bash
cat /etc/group
```
# Crear directorio
```bash
sudo mkdir -p /sysadmin/data
```
# Asignarle usuario y grupo
```bash
sudo chown steve:sysadmin /sysadmin/data
ls -la /sysadmin/
```
# Configurar permisos
```bash
chmod 2770 /sysadmin/data
```
# Verificar
```bash
ls -ld /sysadmin/data
```