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
/opt/mssql-tools18/bin/sqlcmd \
  -S datacenter-server-28788.database.windows.net \
  -U datacenter-admin \
  -P "" \
  -d datacenter-sqldb \
  -C
```

Debemos ejecutar linea por linea:
```
CREATE TABLE Inventario1 (
    Id INT PRIMARY KEY IDENTITY(1,1),
    NombreAsset VARCHAR(50) NOT NULL,
    Estado VARCHAR(20),
    FechaRegistro DATETIME DEFAULT GETDATE()
);
go
```

```
INSERT INTO Inventario1 (NombreAsset, Estado)
VALUES ('Servidor-XFusion-01', 'Online');
go
```

```
SELECT * FROM Inventario1;
go
```