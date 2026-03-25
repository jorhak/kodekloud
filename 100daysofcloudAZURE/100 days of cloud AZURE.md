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
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
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
|Username|[kk_lab_user_main@azurefree.onmicrosoft.com]|
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

# Day 9: Attach Network Interface Card (NIC) to Azure Virtual Machine
```
The devops DevOps team is migrating services to Azure. They are breaking down tasks to ensure better control and optimization. You are tasked with attaching an existing network interface (NIC) to a virtual machine (VM).

An existing VM named devops-vm and a network interface named devops-nic already exist in the westus region.

Attach the network interface devops-nic to the VM devops-vm.
Ensure the NIC's status is attached before submitting the task.
Make sure that the virtual machine initialization has been completed before submitting this task.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Wed Jan 28 18:28:29 UTC 2026
End Time	Wed Jan 28 19:28:29 UTC 2026
```

Lo primero que vamos hacer sera listar **Resource Groups**:
```
az group list
```

Capturamos el **name**:
```
RG_NAME=$(az group list --query [].name --output tsv)
```

Listar las **VM** de este **Resource Groups**:
```
az vm list -d --resource-group $RG_NAME
```
Buscamos _networkProfile>networkInterfaces>id_. Podemos ver que esta asociado a **devops-vmVMNic**.

Capturamos su **id** y **name**:
```
VM_ID=$(az vm list -d --resource-group $RG_NAME --query [].id --output tsv)
VM_NAME=$(az vm list -d --resource-group $RG_NAME --query [].name --output tsv)
```

Listar **NIC**:
```
az network nic list --resource-group $RG_NAME
```
Podemos ver que tenemos dos **NIC**: devops-nic y devops-vmVMNic. Debemos obtener el nombre de la primer **NIC**.
Capturar **NIC**:
```
NIC_NAME=$(az network nic list --resource-group $RG_NAME --query [0].name --output tsv)
```

Vamos a detener la **VM**:
```
az vm stop --ids $VM_ID
```

Asignar **NIC**:
```
az vm nic set --resource-group $RG_NAME --vm-name $VM_NAME --nics $NIC_NAME --primary-nic $NIC_NAME
```

Reiniciar **VM**:
```
az vm start --ids $VM_ID
```

Para verificar que se realizo la asignacion ejecutamos:
```
az vm list -d --resource-group $RG_NAME
```

# Day 10: Attach Public IP to Azure Virtual Machine
```
The Nautilus DevOps team has already set up a virtual machine and allocated a public IP address. The final task is to attach this public IP to the VM's network interface card (NIC).

An existing VM named devops-vm-pip and a public IP address named devops-pip already exist.

Attach the public IP devops-pip to the network interface of the VM devops-vm-pip.
Make sure the VM is properly assigned the public IP.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Wed Jan 28 19:18:18 UTC 2026
End Time	Wed Jan 28 20:18:18 UTC 2026
```

Lo primero que vamos hacer es listar **Resource Groups**:
```
az group list
```

Captura **name** de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
En este grupo de recursos se van a tener los recursos necesarios para realizar el lab.

Debemos listar las **VM** que se encuentran en este **Resource Groups**:
```
az vm list -d --resource-group $RG_NAME
```
Podemos ver que **VM** no tiene IP publica.

Capturar el _id_ y _name_ de **VM**:
```
VM_ID=$(az vm list --resource-group $RG_NAME --query [0].id --output tsv)
VM_NAME=$(az vm list --resource-group $RG_NAME --query [0].name --output tsv)
```

Detener **VM**:
```
az vm stop --ids $VM_ID
```

Listar **NIC**:
```
az network nic list --resource-group $RG_NAME
```
Esta es la interfaz que tiene configurada nuestra **VM** a la que se le debe asignar la **IP Publica**.

Capturar _name_ de **NIC** y _name_ de **ipConfigurations**:
```
NIC_NAME=$(az network nic list --resource-group $RG_NAME --query [0].name --output tsv)
IP_CONFIG_NAME=$(az network nic list --resource-group $RG_NAME --query [0].ipConfigurations[0].name --output tsv)
```
**NIC_NAME** es el nombre de la interfaz de red y **IP_CONFIG_NAME** es el nombre de la configuracion que es parte de la interfaz de red.

Listar **IP Public**:
```
az network public-ip list
```

Obtener _id_ y _name_ de **IP Public**:
```
PIA_ID=$(az network public-ip list --resource-group $RG_NAME --query [0].id --output tsv)
PIA_NAME=$(az network public-ip list --resource-group $RG_NAME --query [0].name --output tsv)
```

Asignar **IP Public** a **NIC**:
```
az network nic ip-config update \
   --name $IP_CONFIG_NAME \
   --nic-name $NIC_NAME \
   --resource-group $RG_NAME \
   --public-ip-address $PIA_NAME
```

Reiniciamos **VM**
```
az vm start --ids $VM_ID
```

Verificar que **VM** ya tiene asignada la **IP Publica**:
```
az vm list -d --resource-group $RG_NAME 
```

# Day 11: Change Azure Virtual Machine Size Using Console
```
The Nautilus Devops team is migrating a portion of their infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team has identified one VM that is underutilized and has decided to change its size to optimize resource usage.

1) Change the VM size from `Standard_B1s` to `Standard_B2s` for the virtual machine named `devops-vm`.

2) Ensure the VM is in the `running` state after the size change is complete.  
  

  

Use the below given **Azure Credentials:** (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Thu Jan 29 19:27:00 UTC 2026|
|End Time|Thu Jan 29 20:27:00 UTC 2026|

  
`Notes:`

- Create the resources only in `eastus` region.
- Make sure the VM is in the `Running` state after resizing.
```

Lo primero que vamos hacer sera listar **Resource Groups**:
```
az group list
```

Capturamos su _name_:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
Una vez tengamos el **Resource Groups** vamos a saber donde esta la **VM** a la que le vamos a cambiar el tamano.

Mostrar los detalles de la **VM** y ver su _size_ que tiene:
```
az vm show -d --resource-group $RG_NAME --name datacenter-vm
```
En _hardwareProfile_>_vmSize_, vemos el **size** que tiene.

Capturar el _id_ y _name_ de **VM** a la que se le va cambiar el tamano:
```
VM_ID=$(az vm list --resource-group $RG_NAME --query [0].id --output tsv)
VM_NAME=$(az vm list --resource-group $RG_NAME --query [0].name --output tsv)
```

Detener **VM**:
```
az vm stop --ids $VM_ID
```

Cambiar tamano de **VM**:
```
az vm resize --size Standard_B2s --ids $VM_ID
```

Reiniciar **VM**:
```
az vm start --ids $VM_ID
```

Verificar el cambio:
```
az vm show -d --resource-group $RG_NAME --name $VM_NAME
```

# Day 12: Add and Manage Tags for Azure Virtual Machines
```
The Nautilus DevOps team is migrating a portion of their infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team has identified one VM that is not tagged properly so they decided to tag it as needed.

Add the tag Environment=dev to the virtual machine named nautilus-vm.



Use the below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Fri Jan 30 13:09:11 UTC 2026
End Time	Fri Jan 30 14:09:11 UTC 2026
```

Lo primero que vamos hacer sera obtener **Resource Groups**:
```
az group list
```

Capturar _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Listar las **VM**:
```
az vm list --resource-group $RG_NAME
```

Capturar _id_ y _name_ de la **VM**:
```
VM_ID=$(az vm list --query [0].id --output tsv)
VM_NAME=$(az vm list --query [0].name --output tsv)
```

Ver detalladamente **VM**:
```
az vm show -d --resource-group $RG_NAME --name $VM_NAME
```
Podemos ver que en _tags_ no tenemos nada.

Detener **VM**:
```
az vm stop --ids $VM_ID
```

Agregar **tag**:
```
az vm update -g $RG_NAME -n $VM_NAME --set tags.Environment=dev
```

Reiniciar **VM**:
```
az vm start --ids $VM_ID
```

Verificar **VM** con el nuevo _tag_:
```
az vm show -d -g $RG_NAME -n $VM_NAME
```
Ahora si vemos que _tag_ tiene lo que acabamos de agregar.

# Day 13: SSH into an Azure Virtual Machine
```
The Nautilus DevOps team is working on setting up secure SSH access for their virtual machines in Azure. One of the requirements is to add the SSH public key of the root user from the Azure client host (landing host) to the `datacenter-vm` Azure VM's `authorized_keys` file. This ensures secure and password-less SSH access to the VM.

### Task Details:

1) **VM Details**:

- The VM is named `datacenter-vm` and is running in the `eastus` region. The default SSH user is `azureuser` — use this user to connect to the VM.
- You need to add the root user's SSH public key from the Azure client host to the `authorized_keys` file of the VM's root user.
- The SSH public key of the root user on the Azure client host is located at `/root/.ssh/id_rsa.pub`.

2) **Public Key Addition**:

- Copy the public key located at `/root/.ssh/id_rsa.pub` on the Azure client host to the `authorized_keys` file of the root user on `datacenter-vm`.
- Ensure that the proper permissions for the `.ssh` folder and `authorized_keys` file are set on the VM.

3) **Verification**:

- After adding the public key, make sure that you are able to SSH into the `datacenter-vm` VM as the `root` user from the Azure client host without needing a password.

### Important Notes:

- Ensure that the VM is up and running before attempting to SSH.
- You may need to adjust the firewall or security group rules for the VM to allow SSH access.

  

Use the following Azure credentials to access the Azure portal:

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Fri Jan 30 13:44:47 UTC 2026|
|End Time|Fri Jan 30 14:44:47 UTC 2026|
```

Listar **Resource Groups**:
```
az group list
```

Capturar _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Listar **VM** que se encuentran en **Resource Groups**:
```
az vm list -g $RG_NAME
```

Capturar _id_ y _name_ de **VM**:
```
VM_ID=$(az vm list -g $RG_NAME --query [0].id --output tsv)
VM_NAME=$(az vm list -g $RG_NAME --query [0].name --output tsv)
```

Ver datalladamente **VM**:
```
az vm show -d --ids $VM_ID
```

Capturar _ip publica_ de **VM**:
```
IP_PUBLICA=$(az vm show -d --ids $VM_ID --query publicIps --output tsv)
```

Ingresar a **VM**:
```
ssh azureuser@$IP_PUBLICA 
```
Ingresa sin inconvenientes

Ingresar a **VM** con usuario _root_:
```
ssh root@$IP_PUBLICA
```
Y nos da este error:
```
Please login as the user "azureuser" rather than the user "root".

Connection to 172.212.187.200 closed.
```

Lo que debemos hacer es ingresar con _azureuser_ y luego cambiar a usuario _root_, y modificar el fichero _/root/.ssh/authorized_keys_ que por defecto tiene un comportamiento de seguridad estandar que bloquea explicitamente el acceso a _root_ y te obliga a entrar como _azureuser_.

```
ssh azureuser@$IP_PUBLICA
sudo -i
```

Editamos el fichero _/root/.ssh/authorized_keys_ 
```
vi /root/.ssh/authorized_keys
```
Borramos todo lo que haiga antes de **ssh-rsa**, por decir: no-port-forwarding,no-agent-forwarding...

```
no-port-forwarding,no-agent-forwarding,no-X11-forwarding,command="echo 'Please login as the user \"azureuser\" rather than the user \"root\".';echo;sleep 10;exit 142"
```

Ahora ingresamos con _root_:
```
ssh root@$IP_PUBLICA
```

NOTA: La configuracion del **host** ya esta configurada:
```
sudo vi /etc/ssh/sshd_config
```

```
PermitRootLogin yes
```

# Day 14: Create and Attach Managed Disks in Azure
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

Create a managed disk with the following requirements:

Name of the disk should be datacenter-disk.

Disk type must be Standard_LRS.

Disk size must be 2 GiB.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Fri Jan 30 18:10:25 UTC 2026
End Time	Fri Jan 30 19:10:25 UTC 2026
```

Listar **Resource Groups**:
```
az group list
```

Capturar _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Crear **disk**:
```
az disk create -g $RG_NAME -n datacenter-disk --sku Standard_LRS --size-gb 2
```

Listar **disk**:
```
az disk list -g $RG_NAME
```

# Day 15: Create and Configure Network Security Group (NSG) in Azure
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a network security group (NSG) with the following requirements:

Name of the NSG should be xfusion-nsg.

Add an inbound security rule named Allow-HTTP for HTTP service on port 80, with the source CIDR range of 0.0.0.0/0.

Add another inbound security rule named Allow-SSH for SSH service on port 22, with the source CIDR range of 0.0.0.0/0.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	
Start Time	Fri Jan 30 18:34:58 UTC 2026
End Time	Fri Jan 30 19:34:58 UTC 2026
```

Listar **Resource Groups**:
```
az group list
```

Capturar _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Crear **Network Security Group** (NSG):
```
az network nsg create \
   --name xfusion-nsg \
   --resource-group $RG_NAME 
```

Crear **security rule**:
```
az network nsg rule create \
--name 'Allow-HTTP' \
--nsg-name xfusion-nsg \
--priority 100 \
--resource-group $RG_NAME \
--direction Inbound \
--access Allow \
--protocol Tcp \
--source-address-prefixes '0.0.0.0/0' \
--source-port-ranges '*' \
--destination-port-ranges 80 \
--description "Permitir trafico http entrante por el puerto 80"
```

```
az network nsg rule create \
--name 'Allow-SSH' \
--nsg-name xfusion-nsg \
--priority 101 \
--resource-group $RG_NAME \
--direction Inbound \
--access Allow \
--protocol Tcp \
--source-address-prefixes '0.0.0.0/0' \
--source-port-ranges '*' \
--destination-port-ranges 22 \
--description "Permitir conexion ssh por el puerto 22"
```

Verificar que se agregaron las reglas:
```
az network nsg show -g $RG_NAME -n xfusion-nsg
```

# Day 16: Create a Private Azure Blob Storage Container
```
As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize private Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named devopsst14186 and a private Blob container named devops-blob-30508 within the storage account.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Fri Feb 06 18:25:03 UTC 2026
End Time	Fri Feb 06 19:25:03 UTC 2026
```

Crear variables de entorno:
```
SA_NAME=devopsst5749
PBC_NAME=devops-blob-23786
```

Lo primero que vamos hacer es listar **Resource Groups**:
```
az group list
```

Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Segundo debemos crear un **Storage Account**:
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME
```

Verificar **Storage Account**:
```
az storage account show \
   --resource-group $RG_NAME \
   --name $SA_NAME
```

Tercero crear **Private Blob Container**:
```
az storage container create \
   --name $PBC_NAME \
   --account-name $SA_NAME \
   --auth-mode login
```

Verificar **Private Blob Container**:
```
az storage container list \
   --account-name $SA_NAME \
   --auth-mode login \
   --query "[?name=='$PBC_NAME']"
```

O
```
az storage container show \
   --name $PBC_NAME \
   --auth-mode login \
   --account-name $SA_NAME
```

# Day 17: Create a Public Azure Blob Storage Container
```
As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize public Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named devopsst4220 and a public Blob container named devops-blob-11214 within the storage account. Make sure anonymous read access for containers and blobs is enabled.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Wed Feb 11 15:15:35 UTC 2026
End Time	Wed Feb 11 16:15:35 UTC 2026
```

Variables de entorno
```
SA_NAME=devopsst4220
PBC_NAME=devops-blob-11214
```

Lo primero listar **Resources Groups**:
```
az group list
```

Capturar el _name_ del **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Segundo crear **Storage account**:
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --allow-blob-public-access true
```

Tercero crear **Public Storage Container**:
```
az storage container create \
   --name $PBC_NAME \
   --account-name $SA_NAME \
   --auth-mode login \
   --public-access container
```

Verificar **Public Storage Container**:
```
az storage container show \
   --name $PBC_NAME \
   --auth-mode login \
   --account-name $SA_NAME \
   --query "properties.publicAccess"
```

# Day 18: Copy Data to an Azure Blob Storage Container
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

# Day 19: Convert Public Azure Blob Container to Private
```
The Nautilus DevOps team has been using Azure Blob Storage to manage their data. Recently, they realized that one of their containers, currently public, needs to be restricted for internal use only. Your task is to convert a public Azure Blob container to private.

Two blob containers named datacenter-container-32445 and datacenter-priv-22200 are available in the centralus region within the storage account datacenterst1575. The datacenter-container-32445 is currently public, and datacenter-priv-22200 is private.

1) Convert the blob container datacenter-container-32445 from public to private while leaving datacenter-priv-22200 unchanged.

2) Make sure the access level for datacenter-container-32445 is set to private with no public access.



Use the below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Fri Feb 20 13:57:54 UTC 2026
End Time	Fri Feb 20 14:57:54 UTC 2026
```

Variables de entorno:
```
BC_NAME=datacenter-container-32445
BCP_NAME=datacenter-priv-22200
SA_NAME=datacenterst1575
REGION=centralus
```

Listar los **Resources Groups**:
```
az group list
```

Obtener _name_:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Listar **Blob Containers**:
```
az storage container list \
   --account-name $SA_NAME \
   --auth-mode login
```
Si estan ambos **Blob Containers**.

Cambiar de publico a privado:
```
az storage container set-permission \
   --name $BC_NAME \
   --account-name $SA_NAME \
   --public-access off
```

# Day 20: Deploy Azure Resources Using ARM Template
```
You are tasked with modifying an ARM template for deploying a virtual network. The current template is located in the /root/arm-templates directory under the filename vnet-deployment-template.json. You need to make the following changes to the template:

Change the name and displayName tag of the virtual network to arm-vnet-nautilus.

Update the addressPrefixes to 192.168.0.0/16.

Add one more tag named Environment with value KKE-nautilus.

After making these changes, you need to deploy the ARM template using the Azure CLI.

Use the following command to find out the resource group to use:

az group list --query '[].name' --output table | grep 'kml'
```

1. Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query '[].name' --output tsv)
```
2. Verificar **Virtual Network**:
```
az network vnet list --resource-group $RG_NAME --output table
```

3. Desplegar template
   - Modificar fichero _vnet-deployment-template.json_:
```
cd arm-templates
vi vnet-deployment-template.json
```

```
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "name": "arm-vnet-nautilus",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2023-11-01",
            "location": "[resourceGroup().location]",
            "tags": {
                "displayName": "arm-vnet-nautilus",
                "Environment": "KKE-nautilus"
            },
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "192.168.0.0/16"
                    ]
                }
            }
        }
    ],
    "outputs": {
    }
}
```

-  Ejecutar comando
```
az deployment group create \
   --resource-group $RG_NAME \
   --template-file vnet-deployment-template.json
```

4. Eliminar **Virtual Network**:
   No ejecutar estos comandos si no son necesarios
```
VN_NAME=$(az network vnet list \
   --resource-group $RG_NAME \
   --query [].name \
   --output tsv)
   
az network vnet delete \
   --resource-group $RG_NAME \
   --name $VN_NAME
```

# Day 21: Assigning Public IP to Virtual Machines
```
The Nautilus DevOps Team has received a new request from the Development Team to set up a new Azure Virtual Machine (VM). This VM will be used to host a new application that requires a stable public IP address. To ensure that the VM has a consistent public IP, a Static Public IP address needs to be associated with it. The VM will be named `devops-vm`, and the Static Public IP will be named `devops-pip`. This setup will help the Development Team to have a reliable and consistent access point for their application.

1. Create an Azure VM named `devops-vm` using any available Ubuntu image, with the VM size `Standard_B1s`.
2. Generate an SSH public key on the `azure-client` host and associate it with the VM for SSH access.
3. Associate a Static Public IP address named `devops-pip` with this VM.
4. Ensure the VM is accessible via SSH using the generated public key.

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Thu Feb 26 13:02:28 UTC 2026|
|End Time|Thu Feb 26 14:02:28 UTC 2026|

  
`Notes:`

- Perform all operations in the `Central US` region.
```

Variables de entorno:
```
VM_NAME=xfusion-vm
VM_SIZE=Standard_B1s
SP_IP_NAME=xfusion-pip
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
USERNAME=azureuser
LOCATION=centralus
```

1. Listar **Resource Groups**:
```
az group list
```

2. Obtener _name_:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

3. Crear IP Publica:
```
az network public-ip create \
    --resource-group $RG_NAME \
    --name $SP_IP_NAME \
    --sku Standard \
    --allocation-method Static \
    --location $LOCATION
```

4. Generar llave SSH
```
ssh-keygen -t rsa -b 4096
```

5. Crear **VM**:
```
az vm create \
    --resource-group $RG_NAME \
    --name $VM_NAME \
    --image "$IMAGE" \
    --size $VM_SIZE \
    --public-ip-address $SP_IP_NAME \
    --location $LOCATION \
    --storage-sku Standard_LRS \
    --ssh-key-values /root/.ssh/id_rsa.pub
```

```
az vm open-port \
    --resource-group "$RG_NAME" \
    --name "$VM_NAME" \
    --port 22 \
    --priority 1001
```

5. Verificar **VM**:
```
az vm show -g $RG_NAME -n $VM_NAME -d --query "{NOMBRE:name,IP_PRIVADA:privateIps,IP_PUBLICA:publicIps,ESTADO:powerState}" --output table
```

6. Verificar **Static IP**:
```
az network public-ip show -n $SP_IP_NAME -g $RG_NAME --query "{NAME:name,IP_PUBLICA:ipAddress}" --output table
```

7. Obtener **Static IP**:
```
IP_PUBLICA=$(az network public-ip show -n $SP_IP_NAME -g $RG_NAME --query ipAddress --output tsv)
```

8. Conectar **SSH**:
```
ssh -i ./ssh/id_rsa $USERNAME@$IP_PUBLICA
```

No se que pasa me da un error de que no se pudo conectar por ssh o que la VM no esta corriendo.
Al  parecer era por el usuario le quite ese argumento (--admin-username) y funciono.

# Day 22: Configuring Instances with User DataDay
```
The Nautilus DevOps Team is working on setting up a new virtual machine (VM) to host a web server for a critical application. The team lead has requested you to create an Azure VM that will serve as a web server using Nginx. This VM will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.

As a member of the Nautilus DevOps Team, your task is to create a VM with the following specifications:

**Instance Name:** The VM must be named `nautilus-vm`.

**Image:** Use any available Ubuntu image to create this VM.

**Custom Script Extension/User Data:** Configure the VM to run a custom script during its launch. This script should:

- Install the Nginx package.
- Start the Nginx service.

**Network Security Group (NSG):** Ensure that the VM allows HTTP traffic on port `80` from the internet.  
  

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Sat Feb 28 13:51:31 UTC 2026|
|End Time|Sat Feb 28 14:51:31 UTC 2026|

  
`Notes:`

- Create the resources only in the `East US` region.
- You may use the default resource group or create a new one if needed.
```

Variables de entorno:
```
VM_NAME=devops-vm
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
NSG_NAME=nsg-vm
LOCATION=eastus
```

1. Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

2. Listar _Network Security Group_ (NSG):
```
az network nsg list --output table
```

2. Crear Network Security Group (NSG):
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

- Obtener NSG_ID:
```
NSG_ID=$(az network nsg show --resource-group $RG_NAME --name $NSG_NAME --query id -o tsv)
```

4. Crear _VM_

- Script de instalacion install.sh

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

- Script de instalacion install.yaml
```
vi install.yaml
```

```
#cloud-config
package_update: true
package_upgrade: true

packages:
  - nginx
runcmd:
  - systemctl enable nginx
  - systemctl start nginx

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
    --custom-data install.yaml \
    --location $LOCATION \
    --generate-ssh-keys
```

3. Obtener _publicIps_
```
IP_ADDRESS=$(az vm show --show-details --resource-group $RG_NAME --name $VM_NAME --query publicIps --output tsv)
```

```
curl -i azureuser@$IP_ADDRESS
```

4. Ir al navegador y ver que este instaldo _NGINX_
   En la barra de navegacion colocamos la **IP**.

5. Ingresar por **SSH**
```
ssh -i .ssh/id_rsa azureuser@$IP_ADDRESS
sudo systemctl status nginx
curl localhost
```

# # Day 23: Automating User Data Configuration Using the CLI 
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

# Day 24: Securing Virtual Machine SSH Access
```
The Nautilus DevOps team needs to set up a new Virtual Machine (VM) on the Azure cloud that can be accessed securely from their landing host (`azure-client`). Follow the steps below to complete this task:

1. **Create an SSH Key**: On the `azure-client` host, check if an SSH key already exists. If it doesn’t exist, create a new SSH key on the `azure-client` host that will be used for password-less SSH access.
    
2. **Create a Virtual Machine**: Use the Azure Portal or Azure CLI to create a new Virtual Machine named `xfusion-vm` in the `westus` region. Set the VM size to **Standard_B1s** and configure the VM with **SSH access** for the `azureuser` account using the newly created SSH key.
    
3. **Configure SSH Access**: Ensure that the SSH key from the `azure-client` host is added to the `azureuser` account on `xfusion-vm`, enabling secure, password-less SSH access from the `azure-client` host.
    
4. **Verify Connectivity**: Test the connection from `azure-client` to `xfusion-vm` using SSH to confirm that password-less access has been set up correctly.
    

Complete these tasks entirely within the Azure Portal or Azure CLI.

  

Use the below given **Azure Credentials:** (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Mon Mar 02 18:15:42 UTC 2026|
|End Time|Mon Mar 02 19:15:42 UTC 2026|

`Notes:`

- Create the resources only in `westus` region.
    
- To `display` or `hide` the terminal of the Azure client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Variables de entorno:
```
VM_NAME=devops-vm
VM_SIZE=Standard_B1s
LOCATION=westus
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
```

1. Crear llave **SSH**:
```
ssh-keygen -t rsa -b 4096
```

2. Crear **VM**:
- Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

- Crear **VM**:
```
az vm create \
    --resource-group $RG_NAME \
    --name $VM_NAME \
    --image $IMAGE \
    --size Standard_B1s \
    --storage-sku Standard_LRS \
    --location $LOCATION \
    --ssh-key-values /root/.ssh/id_rsa.pub
```

- Capturar **IP Publica**:
```
IP_ADDRESS=$(az vm show --show-details --resource-group $RG_NAME --name $VM_NAME --query publicIps --output tsv)
```

3. Verificar **NSG**:
```
az network nsg list
```

4. Ver reglas
```
az network nsg rule list \
   -g $RG_NAME \
   --nsg-name devops-vmNSG
```

5. Verificar conexion por **SSH**:
```
ssh -i .ssh/id_rsa azureuser@$IP_ADDRESS
```

6. Estado de **VM**:
```
az vm show -g $RG_NAME -n $VM_NAME -d --query "{NOMBRE:name,IP_PRIVADA:privateIps,IP_PUBLICA:publicIps,ESTADO:powerState}" --output table
```

# Day 25: Expanding and Managing Disk Storage
```
The Nautilus DevOps team needs to expand the storage capacity of an existing virtual machine and add an additional data disk to support increased workloads. This task requires resizing the existing VM disk and mounting a new data disk to the VM.

As a member of the team, perform the following steps:

1) Expand the existing VM xfusion-vm disk from 32Gi to 64Gi.

2) Also create a new standard HDD data disk named xfusion-disk of 64Gi and mount the disk to VM xfusion-vm at location /mnt/xfusion-disk.




Use the below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Tue Mar 03 13:19:07 UTC 2026
End Time	Tue Mar 03 14:19:07 UTC 2026
```

Variables de entorno:
```
VM_NAME=xfusion-vm
DISK_SIZE=64
DISK_NAME=xfusion-disk
LOCATION=/mnt/xfusion-disk
SKU=Standard_LRS
```

1. Obtener _name_ de **Reseource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

2. Ver estado de **VM**:
Vamos a ver el tamano que tiene actualmente y el nombre del disco
```
az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   --query "storageProfile.osDisk.{SIZE:diskSizeGb,NAME_DISK:name}" \
   --output table 
```

3. Detener **VM**
```
az vm stop \
   --resource-group $RG_NAME \
   --name $VM_NAME
```

Mejor opcion, con la anterior podemos seguir facturando.
```
az vm deallocate --resource-group "$RG_NAME" --name "$VM_NAME"
```

- Estado de **VM**:
Vamos a ver el estado despues de apagar la **VM**
```
az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   -d \
   --query "{Name:name,State:powerState}" \
   --output table
```

3. Expandir disco:
- Obtener _name_ del disco que utiliza **VM**:
```
NAME_DISK=$(az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   --query "storageProfile.osDisk.name" \
   --output tsv)
```

- Actualizar tamano de disco:
```
az disk update \
   --resource-group $RG_NAME \
   --name $NAME_DISK \
   --size-gb $DISK_SIZE
```

4. Reiniciar **VM**:
```
az vm start \
   --resource-group $RG_NAME \
   --name $VM_NAME
```
Volvemos a ejecutar el comando del punto 2.

5. Crear y adjuntar **Standart HDD disk**:
- Detener **VM**:
```
az vm deallocate --resource-group "$RG_NAME" --name "$VM_NAME"
```

- Crear y adjuntar
```
az vm disk attach \
   --resource-group $RG_NAME \
   --vm-name $VM_NAME \
   --name $DISK_NAME \
   --sku $SKU \
   --size-gb $DISK_SIZE \
   --lun 0 \
   --new
```

- Reiniciar **VM**
```
az vm start \
   --resource-group $RG_NAME \
   --name $VM_NAME
```

- Ver estado de **VM**:
```
az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   -d \
   --query "{Name:name,State:powerState}" \
   --output table
```

6. Montar **Disco en /mnt/nautilius-disk
```
az vm run-command invoke \
  -g $RG_NAME \
  -n $VM_NAME \
  --command-id RunShellScript \
  --scripts "
    # 1. Esperar a que el sistema reconozca el disco
    sleep 5
    
    # 2. Formatear solo si no tiene sistema de archivos (seguridad)
    sudo mkfs -t ext4 /dev/disk/azure/scsi1/lun0
    
    # 3. Crear el directorio solicitado
    sudo mkdir -p /mnt/xfusion-disk
    
    # 4. Montar el disco
    sudo mount /dev/disk/azure/scsi1/lun0 /mnt/xfusion-disk
    
    # 5. Hacerlo persistente en el fstab para que sobreviva a reinicios
    echo '/dev/disk/azure/scsi1/lun0 /mnt/xfusion-disk ext4 defaults,nofail 1 2' | sudo tee -a /etc/fstab
  " \
  --query "value[0].message" \
  --output tsv
```

7. Verificar
```
az vm run-command invoke \
  --resource-group "$RG_NAME" \
  --name "$VM_NAME" \
  --command-id RunShellScript \
  --scripts "df -h" \
  --query "value[0].message" \
  --output tsv
```

# Day 26: Deploying Virtual Machines in a Public Virtual Network
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

# Day 27: Deploying Virtual Machines in a Private Virtual Network
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

# Day 28: Troubleshooting Public Virtual Network Configurations
```
The Nautilus DevOps Team deployed an Nginx server on an Azure VM in a public VNet named nautilus-vnet. However, the server is still inaccessible from the internet.

As a DevOps team member, complete the following tasks:

Verify VNet Configuration: Ensure nautilus-vnet allows internet access.
Attach Public IP: A public IP named nautilus-pip already exists. Attach this public IP to the VM nautilus-vm to make it accessible from the internet.
Ensure Accessibility: Confirm the VM nautilus-vm is accessible on port 80.
Use the provided Azure credentials to troubleshoot and resolve the issue.


Use the following Azure Credentials: (Run showcreds on azure-client to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Fri Mar 20 20:34:18 UTC 2026
End Time	Fri Mar 20 21:34:18 UTC 2026

Notes:

Create resources only in the East US region.
Ensure the Network Security Group (NSG) is attached to the VM's NIC or subnet and configured to allow HTTP traffic on port 80.
```

Variables de entorno:
```
VNET_NAME=datacenter-vnet
PIP_NAME=datacenter-pip
VM_NAME=datacenter-vm
LOCATION=eastus
```

1. Obtener **name** de **Resource Group**
```
RG_NAME=$(az group list --query [].name --output tsv)
```

2. Verificar **VM**
```
az vm show -d -g $RG_NAME -n $VM_NAME
```

- Obtner _id_ de _Network Interfaces_
```
NIC_ID=$(az vm show -d -g $RG_NAME -n $VM_NAME \
--query "networkProfile.networkInterfaces[0].id" \
--output tsv)
```

- Obtener _name_ de _Network Interfaces_
```
NIC_NAME=$(az network nic list \
   --query [0].name \
   --output tsv)
```

- Obtener _id_ de _Network Security Group_
```
NSG_ID=$(az network nic show --ids $NIC_ID \
--query networkSecurityGroup.id \
--output tsv)
```

- Obtener _name_ de _Network Security Group_
```
NSG_NAME=$(az network nsg list \
--query [].name --output tsv)
```

- Listar permisos de **Network Security Group**
``` 
az network nsg rule list \
   --resource-group $RG_NAME \
   --nsg-name $NSG_NAME
```

- Agregar regla
```
az network nsg rule create \
  --nsg-name $NSG_NAME \
  --resource-group $RG_NAME \
  --name "AllowHTTP" \
  --protocol tcp \
  --direction Inbound \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 80 \
  --access Allow \
  --priority 100
```

3. Verificar **VNET** y ROUTE TABLE
```
az network vnet show -g $RG_NAME -n $VNET_NAME
```

- Obtener _id_ de **routeTable**:
```
RT_ID=$(az network vnet show \
-g $RG_NAME \
-n $VNET_NAME \
--query subnets[0].routeTable.id \
--output tsv)
```

- Verificar route table
```
az network route-table show \
--ids $RT_ID
```

- Obtener _name_ de **route-table**
```
RT_NAME=$(az network route-table show \
--ids $RT_ID \
--query name \
--output tsv)
```

- Obtener _name_ de _SUBNET_
```
SUBNET_NAME=$(az network vnet show \
   -g $RG_NAME \
   -n $VNET_NAME \
   --query subnets[0].name \
   --output tsv)
```

4. Verificar **Public IP**
```
az network public-ip show -g $RG_NAME -n $PIP_NAME
```

5. Adjuntar **IP Publica** a **NIC**
- Obtener IP Configuration
```
IP_CONFIG_NAME=$(az network nic ip-config list \
   --nic-name $NIC_NAME \
   --resource-group $RG_NAME \
   --query [].name \
   --output tsv)
```

```
az network nic ip-config update \
   -n $IP_CONFIG_NAME \
   -g $RG_NAME \
   --nic-name $NIC_NAME \
   --public-ip-address $PIP_NAME
```

6. Verificar **VM**
```
az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   -d \
   --query "{NAME:name,STATE:powerState,IP_Private:privateIps,IP_Public:publicIps}" \
   --output table
```

7. Verificar que **NSG** esta asociado a **Subnet**
```
az network vnet subnet show \
   -g "$RG_NAME" \
   -n "$SUBNET_NAME" \
   --vnet-name "$VNET_NAME" \
   --query networkSecurityGroup.id \
   -o tsv
```

8. Asociar **NSG** a **Subnet**
```
az network vnet subnet update \
  --resource-group $RG_NAME \
  --vnet-name $VNET_NAME \
  --name $SUBNET_NAME \
  --network-security-group $NSG_NAME
```

9. Configurar route-table
```
az network route-table route update \
    --resource-group "$RG_NAME" \
    --route-table-name "$RT_NAME" \
    --name "Block-Internet" \
    --next-hop-type Internet
```

9. Ingresar a **VM**
```
ssh azureuser@IP_PUBLICA
```

10. Instalar **Nginx**
```
sudo apt -y install nginx
```

11. Verificar estado de Nginx
```
curl -I IP_PUBLICA
sudo systemctl status nginx
ps aux | grep nginx
ss -nltp
```




```
az network route-table route list -g MyResourceGroup --route-table-name MyRouteTable
```

Route table 'devops-rtb' does not have a route to the internet.

Nginx server is not reachable at http://20.237.177.114.
opcional
```
az vm user update --resource-group $RG_NAME --name $VM_NAME --username azureuser --ssh-key-value ~/.ssh/id_rsa.pub
```

```
az network route-table route show --ids $ID_ROUTE_TABLE
```
