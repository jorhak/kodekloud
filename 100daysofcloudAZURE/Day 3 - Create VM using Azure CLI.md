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
