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