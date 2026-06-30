```
The Nautilus DevOps team is developing a simple 'To-Do' application using Azure Table Storage to store and manage tasks efficiently. The team needs to create an Azure Table to hold tasks, each identified by a unique taskId. Each task will have a description and a status, which indicates the progress of the task (e.g.,
'completed' or 'in-progress').

Your task is to:

Create an Azure Storage Account named devopstablest31837 with a Table Storage table called tasks. Insert the following tasks into the table:
Task 1: PartitionKey: 'tasks', RowKey: '1', description: 'Learn Table Storage', status: 'completed'
Task 2: PartitionKey: 'tasks', RowKey: '2', description: 'Build To-Do App', status: 'in-progress'
Verify that Task 1 has a status of 'completed' and Task 2 has a status of 'in-progress'.
Note: Use the Azure CLI to insert these tasks into the table.

Use the Azure Portal URL and login credentials below:

Console URL https://portal.azure.com/azurefreekmlprod.onmicrosoft.com
Username kk_lab_user_main@azureonmicrosoft.com
Password contra
Start Time Fri Jun 26 15:21:11 UTC 2026
End Time Fri Jun 26 16:21:11 UTC 2026

Notes:

Create the resources only in the eastus region.
```
# Variables de entorno
```
SA_NAME=nautilustablest20684
TABLE_NAME=tasks
LOCATION=eastus
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
   --query [0].name \
   --output tsv)
```
# Crear Storage Account
```
az storage account create \
--name $SA_NAME \
--resource-group $RG_NAME \
--location $LOCATION \
--sku Standard_LRS
```
# Obtener connectionString
```
CONN_STRING=$(azstorage account show-connection-string \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --query connectionString -o tsv)
```
# Crear table
```
az storage table create \
   --name $TABLE_NAME \
   --connection-string $CONN_STRING \
   --auth-mode login
```
# Ingresar datos
```
az storage entity insert \
   --table-name $TABLE_NAME \
   --connection-string $CONN_STRING \
   --auth-mode login \
   --entity \
   PartitionKey="tasks" \
   RowKey="1" \
   description="Learn Table Storage" \
   status="completed"
```

```
az storage entity insert \
   --table-name $TABLE_NAME \
   --connection-string $CONN_STRING \
   --auth-mode login \
   --entity \
   PartitionKey="tasks" \
   RowKey="2" \
   description="Build To-Do App" \
   status="in-progress"
```
# Verificar
```
az storage entity query \
   --table-name $TABLE_NAME \
   --connection-string $CONN_STRING \
   --auth-mode login
```

```
az storage table list \
   --account-name $SA_NAME \
   --connection-string $CONN_STRING \
   --auth-mode login
```
# Nota:
Aqui puedo quitar auth-mode ya que cuento con connection-string, es por eso que me da el warning que no se toma en cuenta a connection-string.