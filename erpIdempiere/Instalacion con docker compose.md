# Crear fichero idempiere.yml
```
nano idempiere.yml
```

```
version: '3.7'

services:
  idempiere:
    image: idempiereofficial/idempiere:12-master
    volumes:
      - idempiere_config:/opt/idempiere/configuration
      - idempiere_plugins:/opt/idempiere/plugins
    environment:
      - TZ='America/La Paz'
    ports:
      - 8080:8080
      - 8443:8443
      - 12612:12612
    depends_on:
      - postgres

  postgres:
    image: postgres:16
    volumes:
      - idempiere_data:/var/lib/postgresql/data
    environment:
      - TZ='America/La Paz'
      - POSTGRES_PASSWORD=postgres
    ports:
      - 5432:5432

volumes:
  idempiere_data:
  idempiere_plugins:
  idempiere_config: 
```

## Ejecutar fichero idempiere.yml
```
docker compose -f idempiere.yml up
```

## Listar contenedores
```
docker compose -f idempiere.yml ps
```

## Verificar
Abrimos el navegador
```
IP:8443/webui
```