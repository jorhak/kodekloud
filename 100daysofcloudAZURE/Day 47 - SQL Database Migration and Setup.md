```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to Azure. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. As part of this migration, they are focusing on setting up and managing Azure SQL Databases, implementing backup processes, and ensuring data recovery. Below are the tasks they require you to perform:

Task 1: Create an Azure SQL Database

Create a publicly accessible Azure SQL Database instance with the following details:

Database Name: devops-sqldb.
Server Name: devops-server-22111.
Location: West US
Backup Storage Redundancy: Locally-redundant backup storage.
Hardware Configuration: Basic (For less demanding workloads).
Admin Username: devops-admin.
Admin Password: Set an appropriate password.
Database Size: Set to 2 GiB.
Keep all other configurations as default.
Ensure the database is in the Ready state.

Task 2: Create a Storage Account
Create a Storage Account named devopsst15510.
Configure a Blob Container named devops-container-11990 within this storage account.

Task 3: Backup the Azure SQL Database
Take a backup of the Azure SQL Database instance devops-sqldb and store it in the Blob Container:
Storage Account: devopsst15510.
Blob Container: devops-container-11990.
Backup File Name: devops-db-backup.
Ensure the backup is fully exported to the blob container.

Task 4: Download the Backup
Download the backup file from the Blob Container to the /opt directory on the azure-client host.
Ensure the file is accessible and properly named based on its extension.
Requirements for Completion
Ensure the SQL Database is in the Ready state.
Confirm the backup is stored in the specified Blob Container.
Verify the backup file is successfully downloaded to the /opt directory on the client host.

Use below given Azure Portal Credentials:

Portal URL https://portal.azure.com
Username kk_lab_user_main@azure.onmicrosoft.com
Password contra
Start Time Sat Jul 04 19:24:40 UTC 2026
End Time Sat Jul 04 20:24:40 UTC 2026
```
# Variables de entorno
```
DB_NAME=datacenter-sqldb
SERVER_NAME=datacenter-server-25422
LOCATION=westus
DB_BSR=Local
DB_HC=Basic
USERNAME=datacenter-admin
PASSWORD='$$44Respaldo$$44'
DB_SIZE=2GB
SA_NAME=datacenterst10953
BC_NAME=datacenter-container-8812
BACKUP_NAME=datacenter-db-backup.bacpac
DEST_FILE=/opt
```
Aqui debemos tener cuidado con la variable PASSWORD ya que tiene caracteres especiales es por eso que utilizamos comillas simple y cuando usamos esta variable la utilizamos con comillas dobles.
# Obtener Resource Group
```
RG_NAME=$(az group list \
     --query [0].name \
     --output tsv)
```
# 1 Crear Server
```
az sql server create \
   --name $SERVER_NAME \
   --resource-group $RG_NAME \
   --admin-password "$PASSWORD" \
   --admin-user $USERNAME \
   --location $LOCATION
```
## 1.1 Crear SQL Database
```
az sql db create \
    --name $DB_NAME \
    --resource-group $RG_NAME \
    --server $SERVER_NAME \
    --backup-storage-redundancy $DB_BSR \
    --edition $DB_HC \
    --max-size $DB_SIZE
```
## 1.2 Verificar estado DB
```
az sql db show \
   --name $DB_NAME \
   --resource-group $RG_NAME \
   --server $SERVER_NAME \
   --query "{Nombre:name, Estado:status, Edicion:edition, TamanoMaximo:maxSizeBytes, RedundanciaBackup:requestedBackupStorageRedundancy}"\
   --output table
```
Nos vamos a paso 8, para validar que la base de datos esta creada y creamos una tabla a la cual le insertamos unas tuplas.
# 2 Crear Storage Account
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku Standard_LRS \
   --allow-blob-public-access true \
   --public-network-access Enabled
```
## 2.1 Obtener Storage Key
```
STORAGE_KEY=$(az storage account keys list \
   --account-name $SA_NAME \
   --resource-group $RG_NAME \
   --query [0].value -o tsv)
```

```
echo "Conexion con Storage Key::: $STORAGE_KEY"
```
## 2.2 Obtener connectionString
```
STORAGE_CONN=$(az storage account show-connection-string \
   --resource-group "$RG_NAME" \
   --name "$SA_NAME" \
   --query connectionString -o tsv)
```

```
echo "Cadena de conexión de Storage Account::: $STORAGE_CONN"
```
# 3 Crear Blob Container
```
az storage container create \
   --name $BC_NAME \
   --connection-string "$STORAGE_CONN" \
   --public-access container
```
# 4 Realizar Backup
## 4.1 URI
Esta uri es donde se va guardar nuestro backup dentro del container
```
STORAGE_URI="https://$SA_NAME.blob.core.windows.net/$BC_NAME/$BACKUP_NAME"
```
## 4.2 Habilitamos IP
Habilitamos las Ips para que los otros servicios internos puedan realizar la exportacion.
```
az sql server firewall-rule create \
  --resource-group "$RG_NAME" \
  --server "$SERVER_NAME" \
  --name "AllowAzureServices" \
  --start-ip-address "0.0.0.0" \
  --end-ip-address "0.0.0.0"
```
## 4.3 Exportar DB
```
az sql db export \
   --server $SERVER_NAME \
   --resource-group $RG_NAME \
   --name $DB_NAME \
   --admin-user $USERNAME \
   --admin-password "$PASSWORD" \
   --storage-key $STORAGE_KEY \
   --storage-key-type StorageAccessKey \
   --storage-uri $STORAGE_URI \
   --no-wait
```
Aqui debemos esperar unos minutos entre 5 y 10 minutos.
# 5 Verificar respaldo
```
az sql db op list \
   --database $DB_NAME \
   --resource-group $RG_NAME \
   --server $SERVER_NAME \
   --query "[?operation == 'ExportDatabase'].{Operacion:operation, Estado:state, Progreso:percentComplete}" \
   --output table
```
# 6 Verificar Backup en Blob Container
```
az storage blob list \
  --container-name "$BC_NAME" \
  --account-name "$SA_NAME" \
  --connection-string "$STORAGE_CONN" \
  --query "[].{Archivo:name, TamanoBytes:properties.contentLength}" \
  -o table
```
# 7 Descargar fichero
```
az storage blob download-batch \
   --destination $DEST_FILE \
   --source $BC_NAME \
   --connection-string $STORAGE_CONN
```
# 8 Crear container para crear tablas y tuplas (OPCIONAL)
## 8.1 Habilitamos IP
Habilitamos la IP de nuestro host de KillerCoda para poder ejecutar los comandos
```
az sql server firewall-rule create \
  --resource-group "$RG_NAME" \
  --server "$SERVER_NAME" \
  --name "PermitirMiIPLocal" \
  --start-ip-address "212.2.242.179" \
  --end-ip-address "212.2.242.179"
```
## 8.2 Creamos contenedor
```
docker pull mcr.microsoft.com/mssql-tools
docker run -it mcr.microsoft.com/mssql-tools
export SERVER_NAME="datacenter-server-25422"
export DB_NAME="datacenter-sqldb"
export USERNAME="datacenter-admin"
export PASSWORD='$$44Respaldo$$44'
```

```
sqlcmd -S "$SERVER_NAME.database.windows.net" -d "$DB_NAME" -U "$USERNAME" -P "$PASSWORD" -Q "CREATE TABLE Usuarios (ID INT PRIMARY KEY, Nombre VARCHAR(50), Email VARCHAR(50));"
```

```
sqlcmd -S "$SERVER_NAME.database.windows.net" -d "$DB_NAME" -U "$USERNAME" -P "$PASSWORD" -Q "INSERT INTO Usuarios (ID, Nombre, Email) VALUES (1, 'Jon DevOps', 'jon@nautilus.com'), (2, 'Carmelo Admin', 'carmelo@nautilus.com');"
```

```
sqlcmd -S "$SERVER_NAME.database.windows.net" -d "$DB_NAME" -U "$USERNAME" -P "$PASSWORD" -Q "SELECT * FROM Usuarios;"
```