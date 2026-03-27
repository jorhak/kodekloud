1. Descargar la imagen
```
docker pull mcr.microsoft.com/mssql/server:2019-latest
```

2. Crear contenedor
```
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<password>" -e "MSSQL_PID=Evaluation" -p 1433:1433  --name sql2025 --hostname sql2025 -d mcr.microsoft.com/mssql/server:2019-latest
```

- Ingresar al contenedor
```
docker exec -it sql2025 bash
```

3. Conectar
```
./opt/mssql-tools18/bin/sqlcmd -S datacenter-server-27228.database.windows.net,1433 -U datacenter-admin -P "#Mm1C#-bashnNtTrR4sS3nNaA1211" -No
```
