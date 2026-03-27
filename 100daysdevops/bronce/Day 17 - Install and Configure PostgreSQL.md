## Ingresar al servidor
```
ssh peter@172.16.239.10
```

## Verificar version de PostgreSQL
```
psql --version
```

## Conexion a la base de datos
```
psql -U postgres
```

## Crear usuario
```
CREATE USER kodekloud_joy WITH CREATEDB CREATEROLE LOGIN PASSWORD 'LQfKeWWxWD' ;
```

## Asignar todos los permisos
```
ALTER USER kodekloud_joy SUPERUSER;
```

## Ver privilegios asignados al usuario
```
/du
```

## Ingresar con usuario
```
psql -U kodekloud_joy -d postgres
```

## Crear base de datos
```
CREATE DATABASE kodekloud_db7;
```

### Asignar usuario a la base de datos
```
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_joy;
```

### Ver los permisos de la base de datos
```
\l
```
