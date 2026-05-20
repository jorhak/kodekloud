```
The Nautilus DevOps team has encountered an issue with an Azure VM named xfusion-vm. They are unable to install any packages on this VM due to connectivity issues. The team needs to identify the root cause of the problem and resolve it to restore normal operations.

Investigate the connectivity issue preventing package installation on the Azure VM xfusion-vm.
Implement a solution to resolve the connectivity issue and restore package installation capabilities on the VM.
Note: The SSH key required to access the Azure VM is already created and added to the VM's authorized keys. You can find the SSH key at /root/.ssh/id_rsa on the azure-client host.


Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azure.onmicrosoft.com
Password	contra
Start Time	Tue May 19 20:33:43 UTC 2026
End Time	Tue May 19 21:33:43 UTC 2026

Notes:

Create the resources only in the East US region.
To display or hide the terminal of the Azure client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
VM_NAME=xfusion-vm
RULE_NAME=xfusion-rule-out-connection
RULE_DESCRIPTION="Permite la salida a internet"
```
# Obtener Resource Group ID
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# Obtener user y IP publica
```
USER=$(az vm show \
   -n $VM_NAME \
   -g $RG_NAME \
   -d \
   --query osProfile.adminUsername \
   --output tsv)
```

```
IP_PUBLIC=$(az vm show \
   -n $VM_NAME \
   -g $RG_NAME \
   -d \
   --query publicIps \
   --output tsv)
```

# Ingresar al servidor
```
ssh $USER@$IP_PUBLIC
```
#### Actualizar reposirotio de paquetes
```
sudo apt update -y
```
Salimos del servidor ya que no tenemos salida a internet
# Configurar regla de salida
#### Obtener NIC_ID
```
NIC_ID=$(az vm show \
   -n $VM_NAME \
   -g $RG_NAME \
   -d \
   --query "networkProfile.networkInterfaces[0].id" \
   --output tsv)
```
#### Obtener NSG_ID
```
NSG_ID=$(az network nic show \
   --ids $NIC_ID \
   --query "networkSecurityGroup.id" \
   --output tsv)
```
#### Obtener NSG_NAME
```
NSG_NAME=$(az network nsg show \
   --ids $NSG_ID \
   --query name \
   --output tsv)
```

```
az network nsg rule create \
   --name $RULE_NAME \
   --description "$RULE_DESCRIPTION" \
   --nsg-name $NSG_NAME \
   --priority 100 \
   --resource-group $RG_NAME \
   --direction Outbound \
   --access Allow \
   --protocol * \
   --source-address-prefixes '*' \
   --source-port-ranges '*' \
   --destination-address-prefixes '*' \
   --destination-port-ranges '*'
```
# Ingresar al servidor
```
ssh $USER@$IP_PUBLIC
```
#### Actualizar reposirotio de paquetes
```
sudo apt update -y
```
#### Instalar paquete
```
sudo apt install nginx -y
```