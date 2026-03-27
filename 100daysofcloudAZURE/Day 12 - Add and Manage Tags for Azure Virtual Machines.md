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