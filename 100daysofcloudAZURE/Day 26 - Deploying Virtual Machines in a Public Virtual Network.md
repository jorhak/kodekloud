```
The Nautilus DevOps Team has received a request from the Networking Team to set up a new public VNet to support a set of public-facing services. This VNet will host various resources that need to be accessible over the internet. As part of this setup, you need to ensure the VNet has public subnets with automatic public IP assignment for resources. Additionally, a new VM will be launched within this VNet to host public applications that require SSH access. This setup will enable the Networking Team to deploy and manage public-facing applications.

Create a public VNet named devops-pub-vnet, and a subnet named devops-pub-subnet under the same, make sure public IP is being auto-assigned to resources under this subnet. Further, create a VM named devops-pub-vm under this VNet. Make sure SSH port 22 is open for this instance and accessible over the internet. Use the Azure portal to complete the task and ensure that SSH access is configured correctly.


Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Wed Mar 04 03:14:12 UTC 2026
End Time	Wed Mar 04 04:14:12 UTC 2026
Notes:

Create the resources only in the East US region.
```

Variables de entorno:
```
LOCATION=eastus
VNET_NAME=nautilus-pub-vnet
SUBNET_NAME=nautilus-pub-subnet
VM_NAME=nautilus-pub-vm
NSG_NAME=nautilus-nsg
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
SIZE=Standard_B1s
SKU=Standard_LRS
USERNAME=azureuser
```

1. Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

2. Crear **NSG** (Network Security Group)
```
az network nsg create \
   --name $NSG_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION
```

3. Crear **Vnet** (Virtual Network):
```
az network vnet create \
   --name $VNET_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --nsg $NSG_NAME \
   --address-prefixes "10.20.0.0/16"
```

4. Crear **Subnet**:
```
az network vnet subnet create \
   --name $SUBNET_NAME \
   --resource-group $RG_NAME \
   --vnet-name $VNET_NAME \
   --nsg $NSG_NAME \
   --address-prefixes "10.20.0.0/24"
```

5. Crear **VM**:
- Crear llaves
```
ssh-keygen -t rsa -b 4096
```

- Crear **VM**
```
az vm create \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --image $IMAGE \
   --nsg $NSG_NAME \
   --location $LOCATION \
   --size $SIZE \
   --storage-sku $SKU \
   --ssh-key-values /root/.ssh/id_rsa.pub
```

6. Abrir puerto
```
az vm open-port \
   --port 22 \
   --name $VM_NAME \
   --nsg-name $NSG_NAME \
   --priority 100 \
   --resource-group $RG_NAME
```

7. Ingresar por **SSH**
- Obtener **IP Publica**
```
IP_ADDRESS=$(az vm show \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   -d \
   --query "publicIps" \
   --output tsv)
```

- Ingresar por **SSH**
```
ssh -i ~/.ssh/id_rsa $USERNAME@$IP_ADDRESS
```
