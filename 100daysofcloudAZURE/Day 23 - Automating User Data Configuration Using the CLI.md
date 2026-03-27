```
The Nautilus DevOps Team is working on setting up a new virtual machine (VM) to host a web server for a critical application. The team lead has requested you to create an Azure VM that will serve as a web server using Nginx. This VM will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.
As a member of the Nautilus DevOps Team, your task is to create a VM using Azure CLI with the following specifications:

Instance Name: The VM must be named xfusion-vm.
Image: Use any available Ubuntu image to create this VM.
Custom Script Extension/User Data: Configure the VM to run a custom script during its launch. This script should:

Install the Nginx package.
Start the Nginx service.
Network Security Group (NSG): Ensure that the VM allows HTTP traffic on port 80 from the internet.

Instructions:
Use Azure CLI commands to set up the VM in the specified configuration.
Ensure the VM is accessible from the internet on port 80.
The Nginx service should be running after setup.

Use the Azure CLI commands to complete the task.

Notes:
Create the resources only in the East US region.
You may use the default resource group or create a new one if needed.
```

Variables de entorno:
```
VM_NAME=xfusion-vm
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
NSG_NAME=xfusion-nsg
LOCATION=eastus
```

1. Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

2. Listar _Network Security Group_:
```
az network nsg list
```

3. Crear _Network Security Group_:
```
az network nsg create \
  --name $NSG_NAME \
  --resource-group $RG_NAME \
  --location $LOCATION
```

- Agregar reglas:
```
az network nsg rule create \
  --name "AllowHTTP" \
  --nsg-name $NSG_NAME \
  --priority 100 \
  --resource-group $RG_NAME \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "0.0.0.0/0" \
  --source-port-ranges "*" \
  --destination-port-ranges 80 \
  --description "Permitir trafico http entrante por el puerto 80"
```

```
az network nsg rule create \
  --name "AllowSSH" \
  --nsg-name $NSG_NAME \
  --priority 200 \
  --resource-group $RG_NAME \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "0.0.0.0/0" \
  --source-port-ranges "*" \
  --destination-port-ranges 22 \
  --description "Permitir trafico ssh entrante por el puerto 22"
```

- Obtener **NSG_ID**:
```
NSG_ID=$(az network nsg show --resource-group $RG_NAME --name $NSG_NAME --query id -o tsv)
```

4. Crear **VM**
- Script de instalacion
```
vi install.sh
```

```
#!/bin/bash
sudo apt-get update
sudo apt-get install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

- Crear **VM**
```
az vm create \
    --resource-group $RG_NAME \
    --name $VM_NAME \
    --image $IMAGE \
    --size Standard_B1s \
    --nsg $NSG_ID \
    --storage-sku Standard_LRS \
    --custom-data install.sh \
    --location $LOCATION \
    --generate-ssh-keys
```

4. Obtener **IP PUBLICA**:
```
IP_ADDRESS=$(az vm show --show-details --resource-group $RG_NAME --name $VM_NAME --query publicIps --output tsv)
```

5. Prueba
- Desde la terminal
```
curl -I $IP_ADDRESS
o
curl $IP_ADDRESS
```

- Desde el navegador
En la barra de busqueda colocamos la **IP**