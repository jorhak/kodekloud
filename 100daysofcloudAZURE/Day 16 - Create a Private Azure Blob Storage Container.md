```
As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize private Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named devopsst14186 and a private Blob container named devops-blob-30508 within the storage account.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Fri Feb 06 18:25:03 UTC 2026
End Time	Fri Feb 06 19:25:03 UTC 2026
```

Crear variables de entorno:
```
SA_NAME=devopsst5749
PBC_NAME=devops-blob-23786
```

Lo primero que vamos hacer es listar **Resource Groups**:
```
az group list
```

Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Segundo debemos crear un **Storage Account**:
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME
```

Verificar **Storage Account**:
```
az storage account show \
   --resource-group $RG_NAME \
   --name $SA_NAME
```

Tercero crear **Private Blob Container**:
```
az storage container create \
   --name $PBC_NAME \
   --account-name $SA_NAME \
   --auth-mode login
```

Verificar **Private Blob Container**:
```
az storage container list \
   --account-name $SA_NAME \
   --auth-mode login \
   --query "[?name=='$PBC_NAME']"
```

O
```
az storage container show \
   --name $PBC_NAME \
   --auth-mode login \
   --account-name $SA_NAME
```
