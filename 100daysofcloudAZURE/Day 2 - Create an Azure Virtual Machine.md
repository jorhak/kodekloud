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
