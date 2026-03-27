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
VNET_NAME=xfusion-vnet
PIP_NAME=xfusion-pip
VM_NAME=xfusion-vm
LOCATION=eastus
USER=azureuser
```

1. Obtener **name** de **Resource Group**
```
RG_NAME=$(az group list --query [].name --output tsv)
```

2. Verificar **VM**
```
az vm show -d -g $RG_NAME -n $VM_NAME
```

3. Obtener _name_ de _Network Interfaces_ de **VM**
	En la salida del codigo anterior vimos que la _VM_ tiene asociada una _NIC_ la que nos permite conectarnos.
```
NIC_NAME=$(az network nic list \
   --query [0].name \
   --output tsv)
```

4. Obtener _name_ de _Network Security Group_
	La _NIC_ tambien tiene asociada un _NSG_, que funciona como firewall para el trafico de red.
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

Ejeuctamos el comando previo para ver que se haya agregado la regla

5. Verificar **VNET** y **ROUTE TABLE**
	De la salida del comando vamos a ver que la _VNET_ esta asociada a un _route table_ de la cual debemos ver su configuracion
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

- Verificar **route table**
	Aqui podemos ver que nuestro _route table_ no nos esta permitiendo tener salida a internet por la propiedad _nextHopType_.
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

- Configurar route-table
	Con esta configuracion vamos a tener salida a internet
```
az network route-table route update \
    --resource-group "$RG_NAME" \
    --route-table-name "$RT_NAME" \
    --name "Block-Internet" \
    --next-hop-type Internet
```

Para verificar vamos a la propiedad _nextHopType_ a cambiado de _None_ a _Internet_.

6. Verificar **Public IP**
```
az network public-ip show -g $RG_NAME -n $PIP_NAME
```

7. Adjuntar **IP Publica** a **NIC**
- Obtener IP Configuration
	El _IP Configuration_ nos va permitir asociar la _IP Publica_ con _NIC_.
```
IP_CONFIG_NAME=$(az network nic ip-config list \
   --nic-name $NIC_NAME \
   --resource-group $RG_NAME \
   --query [].name \
   --output tsv)
```

- Asociar _IP Publica_ con _NIC_
```
az network nic ip-config update \
   -n $IP_CONFIG_NAME \
   -g $RG_NAME \
   --nic-name $NIC_NAME \
   --public-ip-address $PIP_NAME
```

8. Verificar **VM**
	Verificamos que la _VM_ tene la _IP Publica_.
```
az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   -d \
   --query "{NAME:name,STATE:powerState,IP_Private:privateIps,IP_Public:publicIps}" \
   --output table
```

- Obtener _IP PUBLICA_
```
IP_PUBLIC=$(az vm show \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   -d \
   --query publicIps \
   --output tsv)
```

9. Verificar que **NSG** esta asociado a **Subnet**
- Obtener _name_ de _SUBNET_
	Previamente debemos conseguir el _name_ de la _Subnet_
```
SUBNET_NAME=$(az network vnet show \
   -g $RG_NAME \
   -n $VNET_NAME \
   --query subnets[0].name \
   --output tsv)
```

- Verificar si la _SUBNET_ esta asociada a un _NSG_
	Si no esta asociada al _NSG_ debemos asociarla como indica el **paso 10**
```
az network vnet subnet show \
   -g "$RG_NAME" \
   -n "$SUBNET_NAME" \
   --vnet-name "$VNET_NAME" \
   --query networkSecurityGroup.id \
   -o tsv
```

10. Asociar **NSG** a **Subnet**
```
az network vnet subnet update \
  --resource-group $RG_NAME \
  --vnet-name $VNET_NAME \
  --name $SUBNET_NAME \
  --network-security-group $NSG_NAME
```

Para verificar que estan asociadas debemos ejecutar el comando previo.

11. Ingresar a **VM**
```
ssh $USER@$IP_PUBLIC
```

12. Instalar **Nginx**
```
sudo apt -y install nginx
```

13. Verificar estado de Nginx
```
sudo systemctl status nginx
ps aux | grep nginx
ss -nltp
```

14. Verificar la conexion
```
curl -I $IP_PUBLIC
```
