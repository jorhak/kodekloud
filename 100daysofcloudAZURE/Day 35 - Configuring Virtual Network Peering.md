```
The Nautilus DevOps team has been tasked with demonstrating the use of VNet Peering to enable communication between two VNets. One VNet will be a private VNet that contains a private Azure VM, while the other will be a public VNet containing a publicly accessible Azure VM.

1) Existing Azure Resources:

Public VM: nautilus-pub-vm is already in the public VNet.
Private VNet and VM: nautilus-priv-vnet and nautilus-priv-vm exist in the private VNet with its subnet: nautilus-priv-subnet.
2) Create VNet Peering:

Create a VNet Peering between the Public VNet and Private VNet.
VNet Peering Name: nautilus-pub-to-priv-peering.
3) Test the Connection:

SSH into the public VM and verify that you can ping the private VM.

Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Thu May 21 14:44:57 UTC 2026
End Time	Thu May 21 15:44:57 UTC 2026
Notes:

Create the resources only in the East US region.
```
# Variables de entorno
Abrir otra terminal
```
VM_PUBLIC_NAME=datacenter-pub-vm
VNET_PRIVATE_NAME=datacenter-priv-vnet
VM_PRIVATE_NAME=datacenter-priv-vm
SUBNET_PRIVATE_NAME=datacenter-priv-subnet
VNET_PEERING_NAME=datacenter-pub-to-priv-peering
VNET_PEERING_NAME_2=datacenter-priv-to-pub-peering
LOCATION=eastus
```
# Obtner el Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# 1 Obtener IP Publica y USER de la VM publica
Esto para poder accedera la VM publica y hacer la prueba de conexion.
```
USER_PUBLIC=$(az vm show \
   -n $VM_PUBLIC_NAME \
   -g $RG_NAME \
   -d \
   --query osProfile.adminUsername \
   --output tsv)
```

```
IP_PUBLIC=$(az vm show \
   -n $VM_PUBLIC_NAME \
   -g $RG_NAME \
   -d \
   --query publicIps \
   --output tsv)
```
# 2 Obtener IP Privada y USER de la VM privada
Esto para poder realizar la prueba desde la VM publica con los datos obtenidos.
```
USER_PRIVATE=$(az vm show \
   -n $VM_PRIVATE_NAME \
   -g $RG_NAME \
   -d \
   --query osProfile.adminUsername \
   --output tsv)
```

```
IP_PRIVATE=$(az vm show \
   -n $VM_PRIVATE_NAME \
   -g $RG_NAME \
   -d \
   --query privateIps \
   --output tsv)
```

Solo ejecutar en la otra terminal que se abrio.
```
echo -e "Usuario Privado:::$USER_PRIVATE \nIP Privada:::$IP_PRIVATE"
```

```
ssh $USER_PUBLIC@$IP_PUBLIC
```

```
ping -c2 <IP_PRIVATE>
```
Volver a la terminal previa y volver a ejecutar todos los comanodos previos.
# 3 Ver todos los recursos
Al ver todos los recursos podemos obtener el nombre de los recursos y sacar los nombres que necesitamos para esta tarea.
```
az resource list \
   --location $LOCATION \
   --resource-group $RG_NAME \
   --query "[*].{NOMBRE:name,TIPO:type}" \
   --output table
```
# 4 Obtener ID VNet Publica
Del comando anterior sacamos un dato que necesitamos para obtener el ID de VNet Publica.
```
VNET_PUBLIC_NAME=datacenter-pub-vnet
```

```
VNET_PUBLIC_ID=$(az network vnet show \
   --name $VNET_PUBLIC_NAME \
   --resource-group $RG_NAME \
   --query "id" \
   --output tsv)
```
## Obtener ID SUBNET Publica
```
SUBNET_PUBLIC_ID=$(az network vnet show \
   --name $VNET_PUBLIC_NAME \
   --resource-group $RG_NAME \
   --query "subnets[0].id" \
   --output tsv)
```
# 5 Obtener ID VNet Privada
Obtenemos el ID de VNet Privada.
```
VNET_PRIVATE_ID=$(az network vnet show \
   --name $VNET_PRIVATE_NAME \
   --resource-group $RG_NAME \
   --query "id" \
   --output tsv)
```
## Obtener ID SUBNET Private
```
SUBNET_PRIVATE_ID=$(az network vnet show \
   --name $VNET_PRIVATE_NAME \
   --resource-group $RG_NAME \
   --query "subnets[0].id" \
   --output tsv)
```
# 6 Crear Peering de la VNet Publica hacia la Privada
```
az network vnet peering create \
   --name $VNET_PEERING_NAME \
   --remote-vnet $VNET_PRIVATE_ID \
   --resource-group $RG_NAME \
   --vnet-name $VNET_PUBLIC_NAME \
   --allow-vnet-access \
   --allow-forwarded-traffic
```
#### Ver estado de Peering Public
Aqui el estado sera **Initiated**
```
az network vnet peering show \
   --name $VNET_PEERING_NAME \
   --resource-group $RG_NAME \
   --vnet-name $VNET_PUBLIC_NAME \
   --query peeringState
```
# 7 Crear Peering de la VNet Privada hacia la Publica
```
az network vnet peering create \
   --name $VNET_PEERING_NAME_2 \
   --remote-vnet $VNET_PUBLIC_ID \
   --resource-group $RG_NAME \
   --vnet-name $VNET_PRIVATE_NAME \
   --allow-vnet-access \
   --allow-forwarded-traffic
```
#### Ver estado de Peering Private
Aqui el estado sera **Connected**, y si volvemos al paso previo _Ver estado de Peering Public_ su estado cambia a **Connected**.
```
az network vnet peering show \
   --name $VNET_PEERING_NAME_2 \
   --resource-group $RG_NAME \
   --vnet-name $VNET_PRIVATE_NAME \
   --query peeringState
```
# 8 Ver las reglas de entrada y salida
### Publica
```
NIC_PUBLIC_ID=$(az vm show \
   --name $VM_PUBLIC_NAME \
   --resource-group $RG_NAME \
   --show-details \
   --query "networkProfile.networkInterfaces[0].id" \
   --output tsv)
```

```
NSG_PUBLIC_ID=$(az network nic show \
   --ids $NIC_PUBLIC_ID \
   --query "networkSecurityGroup.id" \
   --output tsv)
```

```
az network nsg show \
   --ids $NSG_PUBLIC_ID
```
### Privada
```
NIC_PRIVATE_ID=$(az vm show \
   --name $VM_PRIVATE_NAME \
   --resource-group $RG_NAME \
   --show-details \
   --query "networkProfile.networkInterfaces[0].id" \
   --output tsv)
```

```
NSG_PRIVATE_ID=$(az network nic show \
   --ids $NIC_PRIVATE_ID \
   --query "networkSecurityGroup.id" \
   --output tsv)
```

```
az network nsg show \
   --ids $NSG_PRIVATE_ID
```

# 9 Copiar llaves a VM Public
```
cd .ssh
scp id_rsa $USER_PUBLIC@$IP_PUBLIC:/home/$USER_PUBLIC/.ssh/
scp id_rsa.pub $USER_PUBLIC@$IP_PUBLIC:/home/$USER_PUBLIC/.ssh/
```
# Verificar
Volvemos a la otra terminal
```
ssh $USER_PUBLIC@$IP_PUBLIC
```

```
ping -c2 -W1 <IP_PRIVATE>
```

```
ssh <USER_PRIVATE>@<IP_PRIVATE>
```