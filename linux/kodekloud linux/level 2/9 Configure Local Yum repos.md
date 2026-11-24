```
The `Nautilus` production support team and security team had a meeting last month in which they decided to use local yum repositories for maintaing packages needed for their servers. For now they have decided to configure a local yum repo on `Nautilus Backup Server`. This is one of the pending items from last month, so please configure a local yum repository on `Nautilus Backup Server` as per details given below.  
  

  

a. We have some packages already present at location `/packages/downloaded_rpms/` on `Nautilus Backup Server`.  
  

b. Create a yum repo named `epel_local` and make sure to set `Repository ID` to `epel_local`. Configure it to use package's location `/packages/downloaded_rpms/`.  
  

c. Install package `samba` from this newly created repo.
```
# Ingresar al servidor
```bash
ssh clint@stbkp01
```
# Ir al directorio
```bash
cd /packages/downloaded_rpms/ && ls
```

```text
glibc-2.34-290.el9.x86_64.rpm
glibc-common-2.34-290.el9.x86_64.rpm
glibc-devel-2.34-290.el9.x86_64.rpm
glibc-gconv-extra-2.34-290.el9.x86_64.rpm
glibc-headers-2.34-290.el9.x86_64.rpm
glibc-langpack-en-2.34-290.el9.x86_64.rpm
glibc-minimal-langpack-2.34-290.el9.x86_64.rpm
jansson-2.14-1.el9.x86_64.rpm
libldb-4.24.5-2.el9.x86_64.rpm
libnetapi-4.24.5-2.el9.x86_64.rpm
libtalloc-2.4.4-1.el9.x86_64.rpm
libtdb-1.4.15-1.el9.x86_64.rpm
libtevent-0.17.1-1.el9.x86_64.rpm
libwbclient-4.24.5-2.el9.x86_64.rpm
repodata
samba-4.24.5-2.el9.x86_64.rpm
samba-client-libs-4.24.5-2.el9.x86_64.rpm
samba-common-4.24.5-2.el9.noarch.rpm
samba-common-tools-4.24.5-2.el9.x86_64.rpm
samba-core-libs-4.24.5-2.el9.x86_64.rpm
samba-dcerpc-4.24.5-2.el9.x86_64.rpm
samba-ldb-ldap-modules-4.24.5-2.el9.x86_64.rpm
samba-libs-4.24.5-2.el9.x86_64.rpm
samba-ndr-libs-4.24.5-2.el9.x86_64.rpm
```
El directorio **repodata** es muy importante para la creacion de nuestro repositorio local, si no existe debemos crearlo ejecutando el comando correspondiente:
```bash
createrepo /packages/downloaded_rpms/
```
# Crear repositorio local
```bash
sudo mkdir -p /etc/yum.repos.d/
```

```bash
sudo tee /etc/yum.repos.d/epel_local.repo <<EOF
[epel_local]
name=epel_local
baseurl=file:///packages/downloaded_rpms/
enabled=1
gpgcheck=0
EOF
```
# Limpiar y listar repositorio local
```bash
sudo yum clean all
sudo yum repolist
```
# Instalar samba
```bash
sudo yum search samba
```

```bash
sudo yum info samba
```

```
Last metadata expiration check: 0:00:23 ago on Tue Nov 24 01:10:25 2026.
Available Packages
Name         : samba
Version      : 4.24.5
Release      : 2.el9
Architecture : x86_64
Size         : 887 k
Source       : samba-4.24.5-2.el9.src.rpm
Repository   : epel_local
Summary      : Server and Client software to interoperate with Windows machines
URL          : https://www.samba.org
License      : GPL-3.0-or-later AND LGPL-3.0-or-later
Description  : Samba is the standard Windows interoperability suite of programs for Linux
             : and Unix.
```

```bash
sudo yum install samba
```
# Verificar
```bash
sudo systemctl status smb
```