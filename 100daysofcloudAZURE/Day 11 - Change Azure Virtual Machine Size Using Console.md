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
