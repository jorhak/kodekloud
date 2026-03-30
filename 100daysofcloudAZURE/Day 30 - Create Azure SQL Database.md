```
The Nautilus Devops team is strategizing the migration of a portion of their infrastructure to Azure. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. Recently, they started working on creating and configuring some database instances on Azure.

For this task, create one `publicly` accessible Azure SQL Database instance along with the following details:

1) The name of the Azure SQL Database must be `datacenter-sqldb`.

2) The server name must be `datacenter-server-26232` under `westus`.

3) The compute + storage configuration should be **Basic (For less demanding workloads)**.

4) The backup storage redundancy should be **Locally-redundant backup storage**.

5) Set the login admin username to `datacenter-admin` and set an appropriate password.

6) Set the database size to **2 GiB**.

7) Keep the rest of the configurations as `default`. Finally, make sure the database is in the `Ready` state before submitting this task.  
  

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Fri Mar 27 19:03:08 UTC 2026|
|End Time|Fri Mar 27 20:03:08 UTC 2026|
```

Variabales de entorno:
```
DB_NAME=datacenter-sqldb
SERVER_NAME=datacenter-server-28788
LOCATION=southcentralus
USERNAME=datacenter-admin
SIZE=2GB
BSR=Local
```

```
PASSWORD="AC0.3,JCK26a@dS0r##L1n0="
```

1. Obtener _name_ de **Resource Group**:
```
RG_NAME=$(az group list \
   --query [0].name \
   --output tsv)
```

2. Crear _server_
```
az sql server create \
   --name $SERVER_NAME \
   --resource-group $RG_NAME \
   --admin-password "$PASSWORD" \
   --admin-user $USERNAME \
   --location $LOCATION
```

```
az sql server wait \
   --exists \
   -g $RG_NAME \
   -n $SERVER_NAME
```

3. Crear _database_
```
az sql db create \
   --name $DB_NAME \
   --resource-group $RG_NAME \
   --server $SERVER_NAME \
   --edition Basic \
   --capacity 5 \
   --backup-storage-redundancy $BSR 
```

4. Verificar
```
az sql db show \
   -n $DB_NAME \
   -g $RG_NAME \
   --server $SERVER_NAME
```

5. Acceso publico
```
az sql server firewall-rule create \
    --resource-group $RG_NAME \
    --server $SERVER_NAME \
    -n "AllowAll" \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 255.255.255.255
```

6. Verificar conexion
```
az sql db show-connection-string \
    --client ado.net \
    --name $DB_NAME \
    --server $SERVER_NAME
```

[[Conectar desde contenedor]]

