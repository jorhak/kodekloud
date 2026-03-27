```
The Nautilus DevOps team is presently immersed in data migrations, transferring data from on-premise storage systems to Azure Blob containers. They have recently received some data that they intend to copy to one of the Blob containers.

A Blob container named `nautilus-blob-23151` already exists in the `eastus` region under the storage account `nautilusst30823`. Copy the file `/tmp/nautilus.txt` to the Blob container `nautilus-blob-23151`.  
  

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Thu Feb 19 14:06:25 UTC 2026|
|End Time|Thu Feb 19 15:06:25 UTC 2026|
```

Variables:
```
BC_NAME=nautilus-blob-23151
SA_NAME=nautilusst30823
SOURCE=/tmp/nautilus.txt
```

Lo primero listar **Resources Groups**:
```
az group list
```

Obtener _name_
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
En este **Resource Group** es el workspace donde vamos a trabajar.

Listar Blob Storage:
```
az storage container show \
   --name $BC_NAME \
   --auth-mode login \
   --account-name $SA_NAME 
```

Copier fichero a **Blob Storage**
```
az storage blob upload \
    --account-name $SA_NAME \
    --container-name $BC_NAME \
    --name nautilus.txt \
    --file $SOURCE \
    --auth-mode login
```
