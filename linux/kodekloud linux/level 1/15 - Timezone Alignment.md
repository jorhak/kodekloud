```
In the daily standup, it was noted that the timezone settings across the Nautilus Application Servers in the Stratos Datacenter are inconsistent with the local datacenter's timezone, currently set to America/Maceio.



Synchronize the timezone settings to match the local datacenter's timezone (America/Maceio).
```

Ingresar a cada servidor
```
ssh tony@172.16.238.10
ssh steve@172.16.238.11
ssh banner@172.16.238.12
```

1. Verificar zona y hora actual:
```
timedatectl
```

2. Listar las zonas disponibles:
```
timedatectl list-timezones | grep America/Maceio
```

3. Configurar zona horaria
```
sudo timedatectl set-timezone America/Maceio
```