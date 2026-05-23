```
The Nautilus DevOps team needs to optimize data retention costs by automating the deletion of old blobs. They plan to implement Blob Lifecycle Management for a specific container in Azure Storage.

### Task:

1) **Create a Storage Account**:

- Name the storage account `datacenterstor4022`.
- Set the **region** to **East US**.
- Use **Locally-redundant storage (LRS)** as the redundancy option.

2) **Create a Blob Container**:

- Name the container `datacenter-container4022`.

3) **Upload a File to the Container**:

- Upload the file named `tempfile.txt` to the container. The file is present under `/root` of the client host.

4) **Configure Blob Lifecycle Management**:

- Apply a Lifecycle Management rule named `datacenter-del-rule` to the container `datacenter-container4022` to delete blobs after `7` days of last modification.

5) **Validation**:

- Verify that the Lifecycle Management rule named `datacenter-del-rule` is correctly applied.

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Fri May 22 14:08:13 UTC 2026|
|End Time|Fri May 22 15:08:13 UTC 2026|

  
`Notes:`

- Create the resources only in the `East US` region.
- Use the Azure Portal or Azure CLI for resource creation.
- Ensure the storage account and container are properly configured.
```
# Variables de entorno
```
SA_NAME=datacenterstor4022
LOCATION=eastus
SKU=Standard_LRS
BC_NAME=datacenter-container4022
UPLOAD_FILE=tempfile.txt
LM_RULE_NAME=datacenter-del-rule
RULE_DAY=7
```
# Obtener Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# 1 Crear Storage Account (SA)
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku $SKU
```
# 2 Crear Blob Container (BC)
```
az storage container create \
   --name $BC_NAME \
   --account-name $SA_NAME \
   --auth-mode login
```
# 3 Subir fichero a Blob Container
```
az storage blob upload \
    --account-name $SA_NAME \
    --container-name $BC_NAME \
    --name $UPLOAD_FILE \
    --file /root/$UPLOAD_FILE \
    --auth-mode login
```
#### Verificar que se subio el fichero
```
az storage blob list \
    --account-name $SA_NAME \
    --container-name $BC_NAME \
    --output table \
    --auth-mode login
```
# 4 Gestionar ciclo de vida
```
vi policy.json
```

```
{
  "rules": [
    {
      "name": "datacenter-del-rule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ]
        },
        "actions": {
          "baseBlob": {
            "delete": { "daysAfterModificationGreaterThan": 7 }
          }
        }
      }
    }
  ]
}
```
#### Crear politica
```
az storage account management-policy create \
   --account-name $SA_NAME \
   --policy @policy.json \
   --resource-group $RG_NAME
```
#### Verificar politica
```
az storage account management-policy show \
  --account-name $SA_NAME \
  --resource-group $RG_NAME
```