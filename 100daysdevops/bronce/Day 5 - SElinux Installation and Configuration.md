```
sudo yum update
sudo yum upgrade

sudo yum install policycoreutils selinux-policy setroubleshoot -y

#Alternativa
#sudo yum install policycoreutils selinux-policy selinux-policy-targeted 
```

## Configurar SELinux mode
Lo que vamos a hacer sera habilitar el modo permisivo o forzado:
- Permisivo: registra las acciones pero no las impone
- Forzado: registra las acciones y las bloquea
```
sudo nano /etc/selinux/config
SELINUX=permissive
```

