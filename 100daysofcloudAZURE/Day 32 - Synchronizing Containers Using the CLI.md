```
As part of a data migration project, the team lead has tasked the team with migrating data from an existing Azure Blob container to a new Blob container. The existing container contains a substantial amount of data that must be accurately transferred to the new container. The team is responsible for creating the new Blob container and ensuring that all data from the existing container is copied or synced to the new container completely and accurately. It is imperative to perform thorough verification steps to confirm that all data has been successfully transferred to the new container without any loss or corruption.

As a member of the Nautilus DevOps Team, your task is to perform the following:

**Create a New Private Azure Blob Container:** Name the container `datacenter-dest-18759` under the storage account `datacenterst1075`.

**Data Migration:** Migrate the file `datacenter.txt` from the existing `datacenter-source-16267` container to the new `datacenter-dest-18759` container.

**Ensure Data Consistency:** Ensure that both containers have the file `datacenter.txt` and confirm the file content is identical in both containers.

**Use Azure CLI:** Use the Azure CLI to perform the creation and data migration tasks.
```

# Variables de entorno
```
BCD_NAME=nautilus-dest-23593
SA_NAME=nautilusst27382
BCS_NAME=nautilus-source-1894
DATA_SYNC_NAME=nautilus.txt
```
# 1 Obtener Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# 2 Crear Blob Container Private, destino
```
az storage container create \
   --name $BCD_NAME \
   --account-name $SA_NAME \
   --auth-mode login \
   --public-acces off
```
# 3 Migration file
```
az storage blob copy start \
   --account-name $SA_NAME \
   --destination-blob $DATA_SYNC_NAME \
   --destination-container $BCD_NAME \
   --source-container $BCS_NAME \
   --source-blob $DATA_SYNC_NAME \
   --auth-mode login
```
# 4 Verificar
### Verificar fichero en los contenedore
```
az storage blob list \
   --container-name $BCS_NAME \
   --account-name $SA_NAME \
   --auth-mode login \
   --query "[?name=='$DATA_SYNC_NAME'].name" \
   --output tsv
```

```
az storage blob list \
   --container-name $BCD_NAME \
   --account-name $SA_NAME \
   --auth-mode login \
   --query "[?name=='$DATA_SYNC_NAME'].name" \
   --output tsv
```
### Verificar integridad
```
az storage blob show \
   --container-name $BCS_NAME \
   --account-name $SA_NAME \
   --name $DATA_SYNC_NAME \
   --auth-mode login \
   --query "properties.contentSettings.contentMd5"
```

```
az storage blob show \
   --container-name $BCD_NAME \
   --account-name $SA_NAME \
   --name $DATA_SYNC_NAME \
   --auth-mode login \
   --query "properties.contentSettings.contentMd5"
```