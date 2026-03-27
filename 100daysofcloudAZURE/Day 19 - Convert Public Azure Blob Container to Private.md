```
The Nautilus DevOps team has been using Azure Blob Storage to manage their data. Recently, they realized that one of their containers, currently public, needs to be restricted for internal use only. Your task is to convert a public Azure Blob container to private.

Two blob containers named datacenter-container-32445 and datacenter-priv-22200 are available in the centralus region within the storage account datacenterst1575. The datacenter-container-32445 is currently public, and datacenter-priv-22200 is private.

1) Convert the blob container datacenter-container-32445 from public to private while leaving datacenter-priv-22200 unchanged.

2) Make sure the access level for datacenter-container-32445 is set to private with no public access.



Use the below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Fri Feb 20 13:57:54 UTC 2026
End Time	Fri Feb 20 14:57:54 UTC 2026
```

Variables de entorno:
```
BC_NAME=datacenter-container-32445
BCP_NAME=datacenter-priv-22200
SA_NAME=datacenterst1575
REGION=centralus
```

Listar los **Resources Groups**:
```
az group list
```

Obtener _name_:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Listar **Blob Containers**:
```
az storage container list \
   --account-name $SA_NAME \
   --auth-mode login
```
Si estan ambos **Blob Containers**.

Cambiar de publico a privado:
```
az storage container set-permission \
   --name $BC_NAME \
   --account-name $SA_NAME \
   --public-access off
```
