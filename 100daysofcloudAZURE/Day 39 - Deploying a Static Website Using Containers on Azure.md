```
The Nautilus DevOps team has been tasked with creating an internal information portal for public access. As part of this project, they need to host a static website on Azure using an Azure Storage account. The Storage account must be configured for public access to allow external users to access the static website directly via the Azure Storage URL.

Task Requirements:

1. Create an Azure Storage account named `xfusionwebst25138` in an existing resource group.
2. Configure the Storage account for static website hosting with `index.html` as the index document.
3. Allow public access to the static website so that the website is publicly accessible.
4. Upload the `index.html` file from the `/root/` directory of the Azure client host to the Storage account's `$web` container.
5. Verify that the website is accessible directly through the Azure Storage static website URL.

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Fri May 29 13:50:55 UTC 2026|
|End Time|Fri May 29 14:50:55 UTC 2026|

  
`Notes:`

- Create the resources only in the `East US` region.
- Use the Azure Storage account's `$web` container to host the static website files.
- To `display` or `hide` the terminal of the Azure client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```
# Variables de entorno
```
SA_NAME=devopswebst22594
LOCATION=eastus
```
# Obtener Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# 1 Crear Storage Account
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku Standard_LRS \
   --kind StorageV2
```
# 2 Configurar Storage Account
```
az storage blob service-properties update \
   --account-name $SA_NAME \
   --index-document index.html \
   --static-website true \
   --auth-mode login
```
# 3 Subir ficheros
```
az storage blob upload-batch \
   --account-name $SA_NAME \
   --destination '$web' \
   --source "/root/" \
   --pattern "index.html" \
   --auth-mode login
```
# Verificar
```
az storage account show \
  --name $SA_NAME \
  --query "primaryEndpoints.web" \
  --output tsv
```

```
az storage account show \
  --name $SA_NAME \
  --resource-group $RG_NAME \
  --query "allowBlobPublicAccess" \
  --output tsv
```