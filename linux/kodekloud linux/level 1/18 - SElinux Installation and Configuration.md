```
Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for App server 2 in the Stratos Datacenter:



Install the required SELinux packages.

Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.

No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.

Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled.
```

1. Ingresar al servidor:
```
ssh steve@172.16.238.11
```

2. Bucar paquetes
```
sudo yum search selinux
``` 

3. Instalar SELinux
```
sudo yum install -y selinux-policy selinux-policy-targeted policycoreutils policycoreutils-python-utils libselinux.x86_64
```

4. Ver estado
```
/usr/sbin/sestatus
o
sestatus
```

5. Deshabilitar SELinux
```
sudo vi /etc/selinux/config
```

```
SELINUX=disabled
```

[[Bootear USB]]