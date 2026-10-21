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