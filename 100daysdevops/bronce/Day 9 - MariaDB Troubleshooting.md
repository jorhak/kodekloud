Aqui tenemos un error ya que MariaDB no esta activo. Esta instalado pero al parecer tiene un error y lo debemos analizar para poder resolverlo.

Se empeso a revisar los logs
```
sudo journalctl -u mariadb.service --since "15 minutes ago"
```

En donde se encontro que no teniamos permisos 
```
ep 10 19:41:14 stdb01.stratos.xfusioncorp.com mariadbd[2712]: 2025-09-10 19:41:14 0 [Warning] Can't create test file '/var/lib/mysql/stdb01.lower-test' (Errcode: 13 "Permission denied")
```

El cual resolvimos con
```
sudo chown -R mysql:mysql /var/lib/mysql
```

Verificar
```
sudo systemctl restart mariadb
sudo systemctl status mariadb
```
