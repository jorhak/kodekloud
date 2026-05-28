```
The Nautilus DevOps team needs to set up an Azure Virtual Machine (VM) to interact with an Azure Blob Storage container for storing and retrieving data. The team must create a private storage account, configure Blob Storage, and test the functionality.

### Task:

1) **Azure Virtual Machine Setup**:

- The VM named `xfusion-vm` already exists in the **East US** region.

2) **Create a Private Storage Account and Blob Container**:

- Create a storage account named `xfusionstor4844` in the **East US** region with **Locally-redundant storage (LRS)**.
- Create a private Blob container named `xfusion-container4844`.

3) **Retrieve Storage Account Key**:

- Get the storage account's access key to configure access for the application.

4) **Create a Test File**:

- SSH into the VM and create a file named `testfile.txt` in the `/home/azureuser` directory with content: "this is a test file".

5) **Upload the File to Blob Storage**:

- Upload the `testfile.txt` file to the Blob container `xfusion-container4844` using the Azure CLI command:
    
    `az storage blob upload --account-name xfusionstor4844 --account-key <access-key> --container-name xfusion-container4844 --name testfile.txt --file /home/azureuser/testfile.txt`
    

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Wed May 27 17:53:34 UTC 2026|
|End Time|Wed May 27 18:53:34 UTC 2026|

  
`Notes:`

- Create the resources only in the `East US` region.
- Use the Azure Portal or Azure CLI for resource creation.
- Ensure the storage account is private and secure.
```
# Variables de entorno
```
VM_NAME=datacenter-vm
LOCATION=eastus
SA_NAME=datacenterstor23337
SKU=Standard_LRS
BC_NAME=datacenter-container23337
```
# Obtener Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# 1 Obtener usuario y IP Publica de VM
```
USER=$(az vm show \
   -n $VM_NAME \
   -g $RG_NAME \
   -d \
   --query osProfile.adminUsername \
   --output tsv)
```

```
IP_PUBLIC=$(az vm show \
   -n $VM_NAME \
   -g $RG_NAME \
   -d \
   --query publicIps \
   --output tsv)
```

```
ssh $USER@$IP_PUBLIC
```
# 2 Crear fichero
```
echo "this is a test file" > testfile.txt
```
# 3 Iniciar sesion
```
az login
```
# 4 Crear Storage Account
Creamos una cuenta de almacenamiento. Por defecto, las cuentas se crean con acceso privado.
Ejecutamos los comandos **Variables de entorno** y **Obtener Resource Group**.
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku $SKU \
   --allow-blob-public-access false \
   --public-network-access Disabled
```
### Obtner Access Key de Storage Account
```
ACCESS_KEY=$(az storage account keys list \
--account-name $SA_NAME \
--resource-group $RG_NAME \
--query [0].value \
--output tsv)
```
# 5 Crear Blob Container
Creamos un contenedor dentro de la cuenta.
```
az storage account show \
   -n $SA_NAME \
   --query networkRuleSet
```

```
az storage account update \
   -n $SA_NAME \
   -g $RG_NAME \
   --allow-blob-public-access true \
   --public-network-access Enabled
```

```
az storage container create \
   --name $BC_NAME \
   --account-name $SA_NAME \
   --auth-mode login
```
# 6 Subir fichero
```
az storage blob upload \
   --account-name $SA_NAME \
   --account-key $ACCESS_KEY \
   --container-name $BC_NAME \
   --name testfile.txt \
   --file /home/azureuser/testfile.txt
```
# 7 Verificar
```
az storage blob list \
   --container-name $BC_NAME \
   --account-name $SA_NAME \
   --account-key $ACCESS_KEY \
   --output table
```

#### Actualizamos para que vuelva a ser privado
Si van a continuar con el siguiente paso **Opcional** omitan este comando.
```
az storage account update \
   -n $SA_NAME \
   -g $RG_NAME \
   --allow-blob-public-access false \
   --public-network-access Disabled
```
# Opcional
#### Generar acceso temporal
```
az storage blob generate-sas \
   --account-name $SA_NAME \
   --account-key $ACCESS_KEY \
   --container-name $BC_NAME \
   --name testfile.txt \
   --permissions r \
   --expiry 2026-05-28T00:00:00Z \
   --full-uri
```

## Prueba
#### Subir imagen
```
az storage blob upload \
   --account-name $SA_NAME \
   --account-key $ACCESS_KEY \
   --container-name $BC_NAME \
   --name image.jpg \
   --file /home/azureuser/image.jpg
```
#### Generar acceso temporal
```
az storage blob generate-sas \
   --account-name $SA_NAME \
   --account-key $ACCESS_KEY \
   --container-name $BC_NAME \
   --name image.jpg \
   --permissions r \
   --expiry 2026-05-28T00:00:00Z \
   --full-uri
```

## Dar permiso a Blob Container
```
az storage container set-permission \
   --account-name $SA_NAME \
   --account-key $ACCESS_KEY \
   --name $BC_NAME \
   --public-access blob
```
#### Prueba
```
https://devopsstor2298.blob.core.windows.net/devops-container2298/image.jpg

https://devopsstor2298.blob.core.windows.net/devops-container2298/testfile.txt
```

# Quitar permisos
```
az storage account update \
   -n $SA_NAME \
   -g $RG_NAME \
   --allow-blob-public-access false \
   --public-network-access Disabled
```

```
az storage container set-permission \
--account-name $SA_NAME \
--account-key $ACCESS_KEY \
--name $BC_NAME \
--public-access off
```

Failed to list blobs in the container.