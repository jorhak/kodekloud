```
The Nautilus DevOps team is expanding their Azure infrastructure and requires the setup of a private Virtual Network (VNet) along with a subnet. This VNet and subnet configuration will ensure that resources deployed within them remain isolated from external networks and can only communicate within the VNet. Additionally, the team needs to provision a Virtual Machine (VM) under the newly created private VNet. This VM should be accessible over SSH from within the VNet only, allowing for secure communication and resource management within the Azure environment.

The name of the VNet must be `datacenter-priv-vnet`, create a subnet named `datacenter-priv-subnet` under the same. Further, create a Virtual Machine named `datacenter-priv-vm` under this VNet. Additionally, create a Network Security Group (NSG) named `datacenter-priv-nsg`, and ensure that the NSG rules for the VM allow access only from within the VNet's CIDR block. Ensure all resources are created in the `Central US` region.

  

Use below given Azure Credentials: (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Sat Mar 07 01:52:06 UTC 2026|
|End Time|Sat Mar 07 02:52:06 UTC 2026|

`Notes:`

- Create the resources **only** in the `Central US` region.
```

Variables de entorno:
```code
LOCATION=centralus
VNET_NAME=nautilus-priv-vnet
VNET_PREFIX=10.0.0.0/16
SUBNET_NAME=nautilus-priv-subnet
SUBNET_PREFIX=10.0.1.0/24
VM_NAME=nautilus-priv-vm
NSG_NAME=nautilus-priv-nsg
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
SIZE=Standard_B1s
SKU=Standard_LRS
USERNAME=azureuser
```

1. Obtener _name_ de **Resource Groups**:
```code
RG_NAME=$(az group list --query [0].name --output tsv)
```

2. Crear **VNet** (Virtual Network)
```code
az network vnet create \
   --name $VNET_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --address-prefixes $VNET_PREFIX
```

3. Crear **Subnet**:
```code
az network vnet subnet create \
   --name $SUBNET_NAME \
   --resource-group $RG_NAME \
   --vnet-name $VNET_NAME \
   --address-prefixes $SUBNET_PREFIX
```

- Verificar **VNet**:
```
az network vnet subnet wait \
   -g $RG_NAME \
   -n $SUBNET_NAME \
   --vnet-name $VNET_NAME \
   --created
```

```
az network vnet list \
   -g $RG_NAME
```

4. Crear **NSG** (Network Security Group)
```code
az network nsg create \
   --name $NSG_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION
```

- Agregar reglas
```code
az network nsg rule create \
  --name "AllowSSHVNet" \
  --nsg-name $NSG_NAME \
  --priority 100 \
  --resource-group $RG_NAME \
  --direction Inbound \
  --access Allow \
  --protocol '*' \
  --source-address-prefixes $VNET_PREFIX \
  --destination-port-ranges 22 \
  --destination-address-prefix $VNET_PREFIX \
  --description "Permitir trafico Interno Vnet CIDR por SSH"
```

- Verificar reglas
```
az network nsg rule list \
   --resource-group $RG_NAME \
   --nsg-name $NSG_NAME 
```

5. Asociar **NSG** a **SubNet**:
```
az network vnet subnet update \
  --resource-group $RG_NAME \
  --vnet-name $VNET_NAME \
  --name $SUBNET_NAME \
  --network-security-group $NSG_NAME
```

6. Crear **VM**:
- Crear llaves
```code
ssh-keygen -t rsa -b 4096
```

- Crear **VM**
```code
az vm create \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --image $IMAGE \
   --nsg $NSG_NAME \
   --vnet-name $VNET_NAME \
   --subnet $SUBNET_NAME \
   --location $LOCATION \
   --size $SIZE \
   --storage-sku $SKU \
   --ssh-key-values /root/.ssh/id_rsa.pub \
   --public-ip-address ""
```

7. Verificar estado de **VM**:

```
az vm wait \
   -g $RG_NAME \
   -n $VM_NAME \
   --created
```

```code
az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   -d \
   --query "{NAME:name,STATE:powerState,IP_Private:privateIps,IP_Public:publicIps}" \
   --output table
```

- Obtener IP Privada:
```code
IP_PRIVADA_VM=$(az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   -d \
   --query "privateIps" \
   --output tsv)
```

8. Crear **VM** JumpServer

- Crear llave SSH
Vamos a cambiarle el nombre _/root/.ssh/id_jump_:
```
ssh-keygen -t rsa -b 4096
```

```
JUMP_VM="xfusion-jump-server"
az vm create \
  --resource-group $RG_NAME \
  --name $JUMP_VM \
  --location $LOCATION \
  --image $IMAGE \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_NAME \
  --nsg $NSG_NAME \
  --ssh-key-values /root/.ssh/id_jump.pub \
  --size $SIZE \
  --storage-sku $SKU \
  --public-ip-address "xfusion-jump-ip"
```

- Verificar JUMP SERVER
```
az vm wait \
   -g $RG_NAME \
   -n $JUMP_VM \
   --created
```

- Borrar **Jump Server**
	Solo si es necesario
```
az vm delete -g $RG_NAME -n $JUMP_VM --force-deletion 1 --yes
```

9. Agregar regla SSH para JumpServer
```
az network nsg rule create \
  --resource-group $RG_NAME \
  --nsg-name $NSG_NAME \
  --name "AllowSSHFromInternet" \
  --priority 150 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --destination-port-ranges 22 \
  --description "Acceso SSH al JumpServer desde Internet"
```

10. Capturar **IP Publica** de Jump Server
```
JUMP_IP=$(az vm show \
   --resource-group $RG_NAME \
   --name $JUMP_VM \
   -d \
   --query publicIps \
   --output tsv)
```

11. SSS Agent Forwarding
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_jump
ssh -A $USERNAME@$JUMP_IP
```

12. Saltar de JUMP SERVER a VM aislada
- Verificar llaves
```
ssh-add -l
```

```
ssh azureuser@10.0.1.4
```
