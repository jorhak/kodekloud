VAMOS A REALIZAR LOS RETOS DE LA NUVE DE AZURE

# Day 1: Create SSH Key Pair for Azure Virtual Machine
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create an SSH key pair with the following requirements:

The name of the SSH key pair should be devops-kp.

The key pair type must be rsa.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Fri Jan 16 14:44:00 UTC 2026
End Time	Fri Jan 16 15:44:00 UTC 2026
```

Vamos a comenzar listando resource group para poder las llaves ahi:
```
az group list
```

Vamos a crear el par de llaves:
```
az sshkey create \
  --name "devops-kp" \
  --resource-group "kml_rg_main-4107ddd6ee5c4b9c" \
  --location "westus" \
  --encryption-type RSA \
  --tags "Environment=Desarrollo" "Owner=Mario Parez"
```

Verificar que se creo el par de llaves
```
az sshkey list
```

# Day 2: Create an Azure Virtual Machine
```
The Nautilus DevOps team is planning to migrate a portion of their infrastructure to the Azure cloud incrementally. As part of this migration, you are tasked with creating an Azure Virtual Machine (VM).

The requirements are:

1) Use the existing resource group.

2) The VM name must be nautilus-vm, it should be in West US region.

3) Use the Ubuntu 22.04 LTS image for the VM.

4) The VM size must be Standard_B1s.

5) Attach a default Network Security Group (NSG) that allows inbound SSH (port 22).

6) Attach a 30 GB storage disk of type Standard HDD.

7) The rest of the configurations should remain as default.

After completing these steps, make sure you can SSH into the virtual machine.


Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Sat Jan 17 14:28:51 UTC 2026
End Time	Sat Jan 17 15:28:51 UTC 2026
```

Primero vamos a listar el _resource group_:
```
az group list
```

```
RESOURCE_GROUP=kml_rg_main-2425f67e659d4fc1
NSG_NAME=primerNSG
LOCATION=westus
IMAGE=Canonical:0001-com-ubuntu-minimal-jammy:minimal-22_04-lts-gen2:latest
VM_NAME=datacenter-vm
```

Segundo vamos a listar el _Network Security Group_
```
az network nsg list --output table
```

Como no hay un **NSG** lo vamos a crear:
```
az network nsg create \
  --name $NSG_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION
```

Y le anadimos reglas:
```
az network nsg rule create \
  --nsg-name $NSG_NAME \
  --resource-group $RESOURCE_GROUP \
  --name "AllowHTTP" \
  --protocol tcp \
  --direction Inbound \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 22 \
  --access Allow \
  --priority 101
```

Obtener el id de **NSG**:
```
NSG_ID=$(az network nsg show --resource-group $RESOURCE_GROUP --name $NSG_NAME --query id -o tsv)
```

Tercero como ya tenemos lo necesario procedemos a crear la VM:
```
az vm create \
    --resource-group $RESOURCE_GROUP \
    --name $VM_NAME \
    --image $IMAGE \
    --size Standard_B1s \
    --nsg $NSG_ID \
    --data-disk-sizes-gb 30 \
    --storage-sku Standard_LRS \
    --admin-username jorhak \
    --assign-identity \
    --generate-ssh-keys 
```

Obtener la **IP**:
```
IP_ADDRESS=$(az vm show --show-details --resource-group $RESOURCE_GROUP --name $VM_NAME --query publicIps --output tsv)
```

```
ssh jorhak@$IP_ADDRESS
```

```

SSH key files '/root/.ssh/id_rsa' and '/root/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM. If using machines without permanent storage, back up your keys to a safe location.
No access was given yet to the 'nautilus-vm', because '--scope' was not provided. You should setup by creating a role assignment, e.g. 'az role assignment create --assignee <principal-id> --role contributor -g kml_rg_main-edeb2a0e59b94e02' would let it access the current resource group. To get the pricipal id, run 'az vm show -g kml_rg_main-edeb2a0e59b94e02 -n nautilus-vm --query "identity.principalId" -otsv'
```

# Day 3: Create VM using Azure CLI
```
The Nautilus DevOps team is in the process of migrating some of their workloads to Azure. One of the tasks involves creating a new Virtual Machine (VM) using the Azure CLI. The team does not have access to the Azure portal but can manage Azure resources via the `azure-client` host (the landing host for this lab).

1) Create a new Azure Virtual Machine named `devops-vm` using the Azure CLI.

2) Use the `Ubuntu2204` image and set the VM size to `Standard_B2s`.

3) Make sure the admin username is set to `azureuser` and SSH keys are generated for secure access.

4) Use `Standard_LRS` storage account, disk size must be `30GB` and ensure the VM `devops-vm` is in the `running` state after creation.
```

Lo primero que vamos hacer sera listar Resource Group
```
az group list
```

Obenemos el nombre y location:
```
RG_NAME=$(az group list --query "[0].name" --output tsv)
LOCATION=$(az group list --query "[0].location" --output tsv)
VM_NAME=devops-vm
USERNAME=azureuser
```

Crear VM:
```
az vm create \
    --resource-group $RG_NAME \
    --name $VM_NAME \
    --image Ubuntu2204 \
    --size Standard_B2s \
    --admin-username $USERNAME \
    --generate-ssh-keys \
    --storage-sku Standard_LRS \
    --data-disk-sizes-gb 30 
```

Obtener IP para conectarnos VM:
```
IP_ADDRESS=$(az vm show --show-details --resource-group $RG_NAME --name $VM_NAME --query publicIps --output tsv)
```

```
ssh $USERNAME@$IP_ADDRESS
```

Borrar VM:
```
az vm delete \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   --force-deletion true
```

# Day 4: Create a Virtual Network (VNet) in Azure
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations.

Create a Virtual Network (VNet) named xfusion-vnet in the westus region with any IPv4 CIDR block.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Tue Jan 20 17:58:50 UTC 2026
End Time	Tue Jan 20 18:58:50 UTC 2026
```

Lo primero que vamos hacer sera ver si hay algun **resource group**:
```
az group list
```

En nuestro caso existe un **resource group** vamos a capturar su _name y location_:
```
RG_NAME=$(az group list --query "[].{name:name}" --output tsv)
REGION=$(az group list --query "[].{location:location}" --output tsv)
VNET_NAME=xfusion-vnet
```

Crear **Virtual Network**:
```
az network vnet create \
   -g $RG_NAME \
   -n $VNET_NAME \
   --address-prefixes 50.0.0.0/24
```

Listar **Virtual Network**:
```
az network vnet list
```

# Day 5: Create a Virtual Network (IPv4) in Azure
```
The Nautilus DevOps team is strategically planning the migration of a portion of their infrastructure to the Azure cloud. Acknowledging the magnitude of this endeavor, they have chosen to tackle the migration incrementally rather than as a single, massive transition. Their approach involves creating Virtual Networks (VNets) as the initial step, as they will be provisioning various services under different VNets.

Create a Virtual Network (VNet) named `nautilus-vnet` in the `westus` region with `192.168.0.0/24` IPv4 CIDR.  
  

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on the `azure-client` host to retrieve credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main-b6948e9e19904f19@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-b6948e9e19904f19@azurefreekmlprod.onmicrosoft.com)|
|Password|UwEEtd&z|
|Start Time|Tue Jan 20 20:30:47 UTC 2026|
|End Time|Tue Jan 20 21:30:47 UTC 2026|
```

Lo primero que vamos hacer sera ver **resource group**:
```
az group list
```

Capturamos su _name_:
```
RG_NAME=$(az group list --query "[].name" --output tsv)
REGION=$(az group list --query "[].location" --output tsv)
VNET_NAME=nautilus-vnet
```

Creamos la **VNet**:
```
az network vnet create \
   -g $RG_NAME \
   -n $VNET_NAME \
   --location $REGION \
   --address-prefixes 192.168.0.0/24
```

Listar **VNet**:
```
az network vnet list
```

# Day 6: Create a Subnet in Azure Virtual Network
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition.

For this task, create a Virtual Network (VNet) named `datacenter-vnet` and one subnet named `datacenter-subnet` within the VNet in the `centralus` region. Make sure the `IPv4 address range` is `10.0.0.0/16`.  
  

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azurefree.onmicrosoft.com](mailto:kk_lab_user_main-faf83903ab1d468f@azurefreekmlprod.onmicrosoft.com)|
|Password|contra|
|Start Time|Wed Jan 21 20:30:52 UTC 2026|
|End Time|Wed Jan 21 21:30:52 UTC 2026|
```

Crear variables:
```
VNET_NAME=datacenter-vnet
SNET_NAME=datacenter-subnet
REGION=centralus
```

Listar **Resource Group**:
```
az group list
```

Capturar _name_ del **Resource Group**:
```
RESOURCE_GROUP_NAME=$(az group list --query "[0].name" --output tsv)
```

Ahora vamos a comenzar con la creacion de **VNet**:
```
az network vnet create \
   --name $VNET_NAME \
   --resource-group $RESOURCE_GROUP_NAME \
   --location $REGION \
   --address-prefixes 10.0.0.0/16
   
```

Seguimos con la **Subnet**:
```
az network vnet subnet create \
   --name $SNET_NAME \
   --resource-group $RESOURCE_GROUP_NAME \
   --vnet-name $VNET_NAME \
   --address-prefixes 10.0.0.0/16
```

# Day 7: Create a Public IP Address for Azure VM
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, allocate a Public IP address, name it as datacenter-pip.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Thu Jan 22 14:29:40 UTC 2026
End Time	Thu Jan 22 15:29:40 UTC 2026
```

Lo primero que debemos hacer es ver **Resource Group**:
```
az group list
```

Capturamos _name_:
```
RG_NAME=$(az group list --query "[0].name" --output tsv)
```

Crear **IP Publica**:
```
IP_PUBLIC_NAME=datacenter-pip
az network public-ip create \
   --name $IP_PUBLIC_NAME \
   --resource-group $RG_NAME \
   --version IPv4
```

Verificar la creacion de la **IP Publica**:
```
az network public-ip list -g $RG_NAME
```

# Day 8: Attach Managed Disk to Azure Virtual Machine
```
The Nautilus nautilus team is migrating services to Azure. They are breaking down tasks to ensure better control and optimization. You are tasked with attaching an existing data disk to a virtual machine (VM).

An existing VM named nautilus-vm and a managed disk named nautilus-disk already exist in the centralus region.

Attach the disk nautilus-disk to the VM nautilus-vm as a data disk.
Ensure the disk is attached to the VM nautilus-vm.
Make sure that the virtual machine initialization has been completed before submitting this task.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Thu Jan 22 15:19:41 UTC 2026
End Time	Thu Jan 22 16:19:41 UTC 2026
```

Lo primero que vamos hacer sera listar **Resource Groups**:
```
az group list
```

Capturar _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query "[0].name" --output tsv)
```
Con este nombre vamos a poder asignar los recursos al **Resource Groups** correcto.

Vamos a listar las **VM**:
```
az vm list
```
Vamos a busar la **VM** que pertenesca al **Resource Groups** previo. **VM** que le vamos a adjuntar su **Disk**

Verificar **Disk**:
```
az vm list --query "[0].storageProfile.osDisk.name" --output tsv
```
Vemos cual es el nombre del **Disk** que tiene asignado nuestra **VM**.

Captuar _id_ y _name_ :
```
VM_ID=$(az vm list --query "[].id" --output tsv)
VM_NAME=$(az vm list --query "[].name" --output tsv)
```
Capturamos el _id_ y _name_ de nuestra **VM** a la que le vamos a cambiar su **Disk**.

Listar los **Disk**
```
az disk list --resource-group $RG_NAME --query "[*].name" --output json
```
Listamos los **Disk** que se encuentran en nuestro **Resource Groups** para comparar con el que tenemos asignado en la **VM**, como tenemos un _disk_ asignado, debemos adjuntar el otro **Disk**.

Capturamos el nombre del otro **Disk**:
```
DISK_NAME=$(az disk list --resource-group $RG_NAME --query "[0].name" --output tsv)
```

Detener **VM**:
```
az vm stop --ids $VM_ID
```

Ver el estatus **VM**:
```
az vm show -d --ids $VM_ID
```
Buscamos _powerState_ para ver si la **VM** esta detenida.

Asignar **Disk**:
```
az vm disk attach \
   --vm-name $VM_NAME \
   --resource-group $RG_NAME \
   --name $DISK_NAME
```

Inicializar **VM**:
```
az vm start --ids $VM_ID
```
