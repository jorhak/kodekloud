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