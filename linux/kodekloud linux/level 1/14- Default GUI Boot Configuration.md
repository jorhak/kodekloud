```
With the installation of new tools on the app servers within the Stratos Datacenter, certain functionalities now necessitate graphical user interface (GUI) access.



Adjust the default runlevel on all App servers in Stratos Datacenter to enable GUI booting by default. It's imperative not to initiate a server reboot after completing this task.
```

Acceder a servidores:
```
ssh tony@172.16.238.10
ssh steve@172.16.238.11
ssh banner@172.16.238.12
```

Ver estado actual de modo de arranque:
```
systemctl get-default
```
Si no da como respuesta _multi-user.target_ debemo cambiarlo.

Cambiar arranque en modo GUI
```
sudo systemctl set-default graphical.target
```