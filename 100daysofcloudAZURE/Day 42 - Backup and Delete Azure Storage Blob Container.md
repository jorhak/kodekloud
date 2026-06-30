```
The Nautilus DevOps team is currently engaged in a cleanup process, focusing on removing unnecessary data and services from their Azure environment. As part of the migration process, several resources were created for one-time use only, necessitating a cleanup effort to optimize their Azure environment.

A private blob container named xfusion-blob-29976 already exists in the eastus region under storage account xfusionst27528.

1) Copy the contents of xfusion-blob-29976 blob container to the /opt directory on the azure-client host (the landing host once you load this lab).
   
2) Delete the blob container xfusion-blob-29976 from the storage account.
   
Use the below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL https://portal.azure.com
Username kk_lab_user_main@azure.onmicrosoft.com
Password contra
Start Time Sun Jun 28 20:12:29 UTC 2026
End Time Sun Jun 28 21:12:29 UTC 2026
```
# Variables de entorno
```
SA_NAME=nautilusst28589
BC_NAME=nautilus-blob-20349
BACKUP_DIR=/opt
```
# Obtener Resource Group
Este comando no es necesario
```
RG_NAME=$(az group list \
   --query [0].name \
   --output tsv)
```
# Crear Backup
```
az storage blob download-batch \
   --destination $BACKUP_DIR \
   --source $BC_NAME \
   --account-name $SA_NAME \
   --auth-mode login
```
# Eliminar Blob Container
```
az storage container delete \
   --name $BC_NAME \
   --account-name $SA_NAME \
   --auth-mode login
```
# Verificar
#### Ver Blob Container
```
az storage container list \
   --account-name $SA_NAME \
   --auth-mode login
```
#### Listar contenido de Blob Container
```
az storage blob list \
   --container-name $BC_NAME \
   --account-name $SA_NAME \
   --auth-mode login
```