```
The `Nautilus` system admins team has rolled out a web UI application for their backup utility on the `Nautilus backup server` within the `Stratos Datacenter`. This application operates on port `8087`, and `firewalld` is active on the server. To meet operational needs, the following requirements have been identified:  
  

  

Allow all incoming connections on port `8087/tcp`. Ensure the zone is set to `public`.
```

1. Ingresar al servidor
```
ssh clint@172.16.238.16
```

2. Habilitar puerto 8087
```
sudo firewall-cmd --permanent --add-port=8087/tcp --zone=public
```

3. Reiniciar
```
sudo firewall-cmd --reload
```

4. Verificar que se agrego
```
sudo firewall-cmd --list-all
```