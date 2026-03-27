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
