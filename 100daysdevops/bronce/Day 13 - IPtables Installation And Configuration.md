## Verificar puerto expuesto
```
curl 172.16.238.10:8086
curl 172.16.238.11:8086
curl 172.16.238.12:8086
curl 172.16.238.14:8086
```

En la ultima **IP** tubimos un incoveniente no nos pudimos conectar, vamos a solucionarlo mas adelante.

## Instalar IPTABLES
### Acceder a los servidores por ssh
```
ssh tony@172.16.238.10
```

## Verificar si esta instalado IPTABLES
```
iptables --version
```

Ya tiene instalada una version que es un legado asi que vamos a instalar una nueva.

## Instalacion de iptables (por si no tienen instalado)
```
sudo dnf install iptables -y
```

## Configurar IPTABLES
```
sudo iptables -I INPUT -p tcp --dport 8086 -j REJECT
sudo iptables -I INPUT -p tcp --dport 8086 -s 172.16.238.14 -j ACCEPT
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

## Asegurar IPTABLES
```
sudo dnf install iptables-services -y
sudo systemctl enable iptables
sudo systemctl start iptables
sudo service iptables save
```

Repetimos todo desde **Instalar IPTABLES** en los otros servidores.
## Para el LBR
### Probar llegada
```
ssh loki@172.16.238.14
```

```
curl 172.16.238.10:8086
curl 172.16.238.11:8086
curl 172.16.238.12:8086
```
