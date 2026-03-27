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
