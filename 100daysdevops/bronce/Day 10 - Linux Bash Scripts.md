### Permisos
Lo primero que hicimos fue configurar la autenticacion sin contrasena ya que el script necesitaba que se ejecuten comandos sin contrasena.
Luego seguimos con la generacion de llaves SSH para enviar el fichero .zip al servidor backup.
Todo esto lo hicimos con la documentacion del dia 7.

Instalar zip
```
sudo dnf install zip -y
```

```
#!/bin/bash
#Script para realizar un backup del directorio
#/var/www/html/news

zip -r xfusioncorp_news.zip /var/www/html/news
mv xfusioncorp_news.zip /backup/
cd /backup
scp xfusioncorp_news.zip clint@172.16.238.16:/backup/
```

## Dar permisos
```
chmod 755 news_backup.sh
```

## Ejecutar script
```
./news_backup.sh
```

## Validar
Abrimos otra terminal y accedemos por SSH a servidor backup y nos dirigimos a la ruta _/backup_ en donde veremos nuestro comprimido _xfusioncorp_news.zip_.