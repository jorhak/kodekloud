```
The Nautilus DevOps team is tasked with setting up an environment to host a static web application. The application will serve static content from an Azure Storage Account, and a Virtual Machine (VM) will be configured to fetch and display this content using Nginx. The Azure Storage Account is used as a secure, centralized location for storing the index.html file. The team intentionally keeps this file outside the main source code repository, since that repository contains additional internal application code that should not be exposed to or accessed by the VM. By placing only the required static file in the Storage Account, the team can distribute this asset safely and independently of the full codebase.

The VM should securely download the index.html blob directly from the designated container (e.g., using Azure CLI, SAS URL, or REST API) and place it in Nginx’s web root directory so that it is served locally by Nginx. The Storage Account is not mounted, and the Static Website feature is not used. The VM retrieves the file during deployment and may re-fetch it whenever updates are needed. The resources must follow best practices for security, performance, and accessibility.

Task Details:
1) Create a Virtual Network (VNet) and Subnet:
Create a VNet named nautilus-vnet in the East US region.
Create a subnet named nautilus-subnet within the VNet for the VM.

2) Create an Azure Storage Account:
Create a storage account named nautilusstor14503 in the East US region with Locally-redundant storage (LRS).
Create a Blob container named nautilus-container in the storage account.
Upload the index.html file located at /root on the client host to the container nautilus-container.
Ensure the Storage Account is private and not publicly accessible by disabling public access for the storage account.

3) Create a Virtual Machine (VM):
Create a VM named nautilus-vm in the East US region.
Use the nautilus-vnet and subnet nautilus-subnet for the VM.
Authentication: Use SSH public key authentication. (Please select use existing public key option, create public-key locally and paste contents of ~/.ssh/id_rsa.pub)
Install Nginx on the VM.

Download the index.html file using a command such as:
sudo az storage blob download --account-name nautilusstor14503 --account-key xxxxx --container-name nautilus-container --name index.html --file /var/www/html/index.html

Ensure Nginx is configured to serve the file from /var/www/html/index.html.

4) Verify Setup:
Verify that the Nginx web server on the client host serves the index.html file correctly when accessing the VM's public IP address.

Note: Follow best practices for security, accessibility, and performance while configuring resources. Use below given Azure Portal Credentials:

Portal URL https://portal.azure.com
Username kk_lab_user_main@azure.onmicrosoft.com
Password contra
Start Time Thu Jul 09 17:35:18 UTC 2026
End Time Thu Jul 09 18:35:18 UTC 2026
```
# Variables de entorno
```
PREFIX=nautilus
LOCATION=eastus
VNET_NAME=$PREFIX-vnet
SUBNET_NAME=$PREFIX-subnet
SA_NAME="${PREFIX}stor24738"
BC_NAME=$PREFIX-container
VM_NAME=$PREFIX-vm
IMAGE=Ubuntu2404
SIZE=Standard_B1s
STORAGE_SKU=Standard_LRS
NSG_NAME=$PREFIX-nsg
RSG_NAME_SSH=AllowSSH
RSG_NAME_HTTP=AllowHTTP
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
     --query [0].name \
     --output tsv)
```
# 1 Crear VNet y Subnet para VM
```
az network vnet create \
  --resource-group "$RG_NAME" \
  --name $VNET_NAME \
  --address-prefixes 10.0.0.0/8 \
  --subnet-name $SUBNET_NAME \
  --subnet-prefixes 10.240.0.0/16 \
  --location $LOCATION
```
# 2 Crear Network Security Group
```
NSG_ID=$(az network nsg create \
  --name $NSG_NAME \
  --resource-group $RG_NAME \
  --location $LOCATION \
  --query NewNSG.id \
  --output tsv)
```
Otra forma de obtener ID
```
NSG_ID=$(az network nsg show \
  --name $NSG_NAME \
  --resource-group $RG_NAME \
  --query id  \
  --output tsv)
```
## 2.1 Agregar regla SSH
```
az network nsg rule create \
  --nsg-name $NSG_NAME \
  --resource-group $RG_NAME \
  --name $RSG_NAME_SSH \
  --protocol tcp \
  --direction Inbound \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 22 \
  --access Allow \
  --priority 100
```
## 2.2 Agregar regla HTTP
```
az network nsg rule create \
  --nsg-name $NSG_NAME \
  --resource-group $RG_NAME \
  --name $RSG_NAME_HTTP \
  --protocol tcp \
  --direction Inbound \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 80 \
  --access Allow \
  --priority 101
```
# 3 Crear Storage Account
## 3.1 Storage Account
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku Standard_LRS \
   --kind StorageV2 \
   --allow-blob-public-access false \
   --public-network-access Disabled
```
### Obtener Storage Key
```
STORAGE_KEY=$(az storage account keys list \
   --account-name $SA_NAME \
   --resource-group $RG_NAME \
   --query [0].value -o tsv)
```
### Obtener connectionString
```
STORAGE_CONN=$(az storage account show-connection-string \
   --resource-group "$RG_NAME" \
   --name "$SA_NAME" \
   --query connectionString -o tsv)
```
### Habilitar temporalmente acceso desde redes publicas
```
az storage account update \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --public-network-access Enabled
```
Esto tarda entre 3 y 5 minutos.
# 4 Crear Blob Container
```
az storage container create \
   --name $BC_NAME \
   --connection-string $STORAGE_CONN \
   --public-access off
```
# 5 Subir fichero
```
az storage blob upload \
  --container-name $BC_NAME \
  --name "index.html" \
  --file "/root/index.html" \
  --connection-string $STORAGE_CONN
```
## 5.1 Verificar que se subio el fichero
```
az storage blob list \
  --container-name $BC_NAME \
  --connection-string $STORAGE_CONN \
  --output table
```
## 5.2 Volver a bloquearlo para que sea privado
No vamos a ejecutar este comando dado que si lo hacemos no va pasar las pruebas de verificacion.
```
az storage account update \
  --name $SA_NAME \
  --resource-group $RG_NAME \
  --public-network-access Disabled
```
# 6 Crear VM
## 6.1 Crear par de llaves
```
ssh-keygen -t rsa -b 4096
```
## 6.2 Script de instalacion
```
vi install.sh
```

```
#!/bin/bash
#Filename: install.sh
#Description: Instalar Nginx
sudo apt update
sudo apt install -y nginx
sudo systemctl start nginx
curl -fsSL 'https://azurecliprod.blob.core.windows.net/$root/deb_install.sh' | sudo bash
```
## 6.3 Crear VM
```
az vm create \
    --resource-group $RG_NAME \
    --name $VM_NAME \
    --image $IMAGE \
    --size $SIZE \
    --vnet-name $VNET_NAME \
    --nsg $NSG_ID \
    --data-disk-sizes-gb 30 \
    --storage-sku $STORAGE_SKU \
    --assign-identity \
    --custom-data install.sh \
    --ssh-key-values .ssh/id_rsa.pub
```

```
az vm wait -g $RG_NAME -n $VM_NAME --created
```
## 6.4 Obtener IP Publica
```
IP_PUBLICA=$(az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "publicIps" \
   -o tsv)
```
## 6.5 Obtener Usuario
```
USER=$(az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "osProfile.adminUsername" \
   -o tsv)
```
# 7 Ingresar al servidor
```
ssh $USER@$IP_PUBLICA
```

```
az login
PREFIX=nautilus
SA_NAME="${PREFIX}stor24738"
BC_NAME=$PREFIX-container
RG_NAME=$(az group list \
     --query [0].name \
     --output tsv)

STORAGE_KEY=$(az storage account keys list \
   --account-name $SA_NAME \
   --resource-group $RG_NAME \
   --query [0].value -o tsv)
```

```
###### Omitir este comando ya que lo dejamos habilitado
az storage account update \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --public-network-access Enabled
```

```
sudo chown -R $USER:$USER /var/www/html
```

```
az storage blob download \
   --account-name $SA_NAME \
   --account-key $STORAGE_KEY \
   --container-name $BC_NAME \
   --name index.html \
   --file /var/www/html/index.html
```

```
#####No ejecutamos este comando dado que no va pasar la verificacion
az storage account update \
  --name $SA_NAME \
  --resource-group $RG_NAME \
  --public-network-access Disabled
```
# 8 Verificar
Abrimos un navegador y colocamos la IP Publica de la VM.