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
