# 2 Group Creation and User Assignment
## Crear grupo
```
sudo groupadd nautilus_noc
```

## Crear usuario y asignar grupo
```
sudo useradd -g nautilus_noc -c "Hola Sonya" -d /home/sonya -s /bin/bash sonya
```

# 3 Linux User Setup with Non-Interactive Shell
```
sudo useradd -s /sbin/nologin john
```

# 4 Service User Creation without Home Directory
```
sudo useradd -M mark
```

# 5 Temporary User Setup with Expiry
```
sudo useradd -e 2024-01-28 -c "Mari Yam" -d /home/mariyam -s /bin/bash mariyam
```

# 6 Linux User Data Transfer
```
Due to an accidental data mix-up, user data was unintentionally mingled on Nautilus App Server 3 at the /home/usersdata location by the Nautilus production support team in Stratos DC. To rectify this, specific user data needs to be filtered and relocated. Here are the details:



Locate all files (excluding directories) owned by user rose within the /home/usersdata directory on App Server 3. Copy these files while preserving the directory structure to the /media directory.
```

## Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Buscar todos los ficheros del propietario mark y copiarlos en la ruta /media
```
sudo find /home/usersdata -type f -user rose -exec cp --parents {} /media/ \;
```
Este comando busca a todos los ficheros que son propiedad del usuario **rose**, lo hace de forma recursiva.
Ahora si solo queremos que solo busque a nivel del padre y no de forma recursiva debemos ejecutar:
```
sudo find /home/usersdata -maxdepth 1 -type f -user mark -exec cp --parents {} /media/ \;
```

# 7 Secure Root SSH Access
```
Following security audits, the xFusionCorp Industries security team has rolled out new protocols, including the restriction of direct root SSH login.



Your task is to disable direct SSH root login on all app servers within the Stratos Datacenter.
```

## Ingresar a servidor
```
ssh tony@172.16.238.10
```

```
ssh steve@172.16.238.11
```

```
ssh banner@172.16.238.12
```
## Configurar SSH
```
sudo vi /etc/ssh/sshd_config
```

Before
```
# Authentication:

#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

After
```
# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

Reiniciar servicio
```
sudo systemctl restart sshd
```

# 8 Data Backup for Developer
```
Within the Stratos DC, the Nautilus storage server hosts a directory named /data, serving as a repository for various developers non-confidential data. Developer mark has requested a copy of their data stored in /data/mark. The System Admin team has provided the following steps to fulfill this request:



a. Create a compressed archive named mark.tar.gz of the /data/mark directory.

b. Transfer the archive to the /home directory on the Storage Server.
```

## Ingresar al servidor
```
ssh natasha@172.16.238.15
```

## Backup el directorio
```
tar czvf mark.tar.gz /data/mark
```

## Mover al directoiro /home
```
mv mark.tar.gz /home/
```

## Restaurar Backup
```
tar xzvf mark.tar.gz
```

# 9 Script Execution Permissions
```
In a bid to automate backup processes, the xFusionCorp Industries sysadmin team has developed a new bash script named xfusioncorp.sh. While the script has been distributed to all necessary servers, it lacks executable permissions on App Server 1 within the Stratos Datacenter.



Your task is to grant executable permissions to the /tmp/xfusioncorp.sh script on App Server 1. Additionally, ensure that all users have the capability to execute it.
```

## Ingresar al servidor
```
ssh tony@172.16.238.10
```

## Modificar permisos
```
sudo chmod 777 /tmp/xfusioncorp.sh
```

# 10 File Permission Correction
```
After conducting a security audit within the Stratos DC, the Nautilus security team discovered misconfigured permissions on critical files. To address this, corrective actions are being taken by the production support team. Specifically, the file named /etc/hosts on Nautilus App 1 server requires adjustments to its Access Control Lists (ACLs) as follows:



1. The file's user owner and group owner should be set to root.

2. Others should possess read only permissions on the file.

3. User kirsty must not have any permissions on the file.

4. User eric should be granted read only permission on the file.
```

## Ingresar al servidor
```
ssh tony@172.16.238.10
```

## Acceder al directorio
```
cd /etc
```

## Listar permisos
```
ls -la hosts
```

## Modificar usuario y grupo a root, habilitar permisos
```
sudo chmod 664 hosts
sudo chown root:root hosts
```

## Bloquear usuario kristy
```
sudo setfacl -m u:kirsty:--- /etc/hosts
```

## Solo lecturar usuario eric
```
sudo setfacl -m u:eric:r-- /etc/hosts
```

## Verificar permisos
```
getfacl /etc/hosts
```

# 11 String Replacement
```
Within the Stratos DC, the backup server holds template XML files crucial for the Nautilus application. Before utilization, these files require valid data insertion. As part of routine maintenance, system admins at `xFusionCorp Industries` employ string and file manipulation commands.  
  

  

Your task is to substitute all occurrences of the string `Text` with `Torpedo` within the XML file located at `/root/nautilus.xml` on the backup server.
```

## Ingresar al servidor
```
ssh clint@172.16.238.16
```

## Ver el contenido
```
sudo head -n 50 /root/nautilus.xml
```

## Modificar contenido
```
sudo sed -i 's/<text>Text<\/text>/<text>Torpedo<\/text>/g' /root/nautilus.xml
```

# 12 Secure Data Transfer
```
A Nautilus developer has stored confidential data on the jump host within Stratos DC. To ensure security and compliance, this data must be transferred to one of the app servers. Given developers lack direct access to these servers, the system admin team has been enlisted for assistance.



Copy /tmp/nautilus.txt.gpg file from jump server to App Server 1 placing it in the directory /home/nfsshare.
```

## Abrir otra terminal
En esta terminal vamos a ingresar a **App Server 1**
```
ssh tony@172.16.238.10
```
## Copiar fichero a otro servidor
Aqui estamos en el servidor **jump server**
```
scp /tmp/nautilus.txt.gpg tony@172.16.238.10:/home/nfsshare
```
## Verificar que se copio en App Server 1
Nos vamos a la terminal que abrimos anteriormente
```
ls -la /home/nfsshare
```