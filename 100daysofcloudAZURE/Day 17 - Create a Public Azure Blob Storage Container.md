```
As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize public Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named devopsst4220 and a public Blob container named devops-blob-11214 within the storage account. Make sure anonymous read access for containers and blobs is enabled.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Wed Feb 11 15:15:35 UTC 2026
End Time	Wed Feb 11 16:15:35 UTC 2026
```

Variables de entorno
```
SA_NAME=devopsst4220
PBC_NAME=devops-blob-11214
```

Lo primero listar **Resources Groups**:
```
az group list
```

Capturar el _name_ del **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Segundo crear **Storage account**:
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --allow-blob-public-access true
```

Tercero crear **Public Storage Container**:
```
az storage container create \
   --name $PBC_NAME \
   --account-name $SA_NAME \
   --auth-mode login \
   --public-access container
```

Verificar **Public Storage Container**:
```
az storage container show \
   --name $PBC_NAME \
   --auth-mode login \
   --account-name $SA_NAME \
   --query "properties.publicAccess"
```
