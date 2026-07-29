```
The Nautilus DevOps team needs to set up an Azure Application Gateway to manage traffic for a backend pool of virtual machines. The gateway will serve as a load balancer, distributing traffic across the VMs.
Task:
1) Azure Virtual Network and Subnet:
Create a Virtual Network (VNet) named devops-vnet in the East US region.
Create a Subnet named devops-subnet within the VNet for the VMs.
Create a Subnet named devops-apgw-subnet within the VNet for the Application Gateway.

2) Azure Virtual Machines:
Create two VMs named devops-vm1 and devops-vm2 in the East US region.
Install Nginx on both VMs.
Configure index.html on VM1 to display "Welcome to KKE Labs:Version 1".
Configure index.html on VM2 to display "Welcome to KKE Labs:Version 2".

3) Azure Application Gateway:
Create an Application Gateway named devops-apgw in the East US region.
Assign the devops-apgw-subnet to the Application Gateway.
Create a frontend IP configuration named devops-apgw-ip.
Add the VMs devops-vm1 and devops-vm2 to the backend pool.
Configure a basic routing rule to distribute traffic between the VMs.

4) Validation:
Verify that the Application Gateway distributes traffic to both VMs.

Ensure that accessing the Application Gateway URL displays either "Welcome to KKE Labs:Version 1" or "Welcome to KKE Labs:Version 2" depending on the load balancing.

Use below given Azure Credentials: (You can run the showcreds command on azure-client host to retrieve these credentials)

Portal URL https://portal.azure.com
Username kk_lab_user_main@azure.onmicrosoft.com
Password contra
Start Time Sat Jul 11 13:39:42 UTC 2026
End Time Sat Jul 11 14:39:42 UTC 2026

Notes:

Create all resources in the East US region.
Use the Azure Portal or Azure CLI for resource creation.
Ensure proper routing and traffic distribution through the Application Gateway.
```
# Variables de entorno
```
PREFIX=devops
IMAGE=Ubuntu2404
SIZE=Standard_B1s
STORAGE_SKU=Standard_LRS
VNET_VM_NAME=$PREFIX-vnet
SUBNET_VM_NAME=$PREFIX-subnet
SUBNET_AGW_NAME=$PREFIX-apgw-subnet
APGW_NAME=$PREFIX-apgw
LOCATION=eastus
IP_AGW_NAME=$PREFIX-apgw-ip
PORT_AGW_NAME=$PREFIX-apgw-port
NSG_NAME=$PREFIX-nsg
RSG_NAME_HTTP=AllowHTTP
ADDRESS_POOL_NAME=$PREFIX-pool
RULE_AGW_NAME=$PREFIX-rule
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
     --query [0].name \
     --output tsv)
```
# 1 Crear VNet y Subnet para VM
```
az network vnet create \
   --name $VNET_VM_NAME \
   --resource-group $RG_NAME \
   --address-prefixes 10.0.0.0/16 \
   --subnet-name $SUBNET_VM_NAME \
   --subnet-prefixes 10.0.1.0/24 \
   --tags Env=DEV Department=IT \
   --location $LOCATION
```
## 1.1 Crear Subnet para Application Gateway
```
az network vnet subnet create \
   --name $SUBNET_AGW_NAME \
   --resource-group $RG_NAME \
   --vnet-name $VNET_VM_NAME \
   --address-prefixes 10.0.2.0/24
```
# 2 Crear Network Security Group
```
NSG_ID=$(az network nsg create \
  --name $NSG_NAME \
  --resource-group $RG_NAME \
  --location $LOCATION \
  --query NewNSG.id \
  --output tsv)
```
Otra forma de obtener ID
```
NSG_ID=$(az network nsg show \
  --name $NSG_NAME \
  --resource-group $RG_NAME \
  --query id  \
  --output tsv)
```
## 2.1 Agregar regla HTTP
```
az network nsg rule create \
  --nsg-name $NSG_NAME \
  --resource-group $RG_NAME \
  --name $RSG_NAME_HTTP \
  --protocol tcp \
  --direction Inbound \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 80 \
  --access Allow \
  --priority 101
```
# 3 Crear VMs
# 3.1 Crear par de llaves
```
ssh-keygen -t rsa -b 4096
```
## 3.2 Script de instalacion
```
vi install-1.sh
```

```
#!/bin/bash
#Filename: install-1.sh
#Description: Instalacion de Nginx

USER=azureuser
sudo apt update
sudo apt install -y nginx
sudo systemctl start nginx
sudo chown -R $USER:$USER /var/www/html
echo "Welcome to KKE Labs:Version 1" > /var/www/html/index.html
```

```
vi install-2.sh
```

```
#!/bin/bash
#Filename: install-2.sh
#Description: Instalacion de Nginx

USER=azureuser
sudo apt update
sudo apt install -y nginx
sudo systemctl start nginx
sudo chown -R $USER:$USER /var/www/html
echo "Welcome to KKE Labs:Version 2" > /var/www/html/index.html
```
## 3.3 Crear VMs
```
for i in 1 2 ; do
   az vm create \
      --resource-group $RG_NAME \
      --name "$PREFIX-vm${i}" \
      --image $IMAGE \
      --size $SIZE \
      --vnet-name $VNET_VM_NAME \
      --subnet $SUBNET_VM_NAME \
      --nsg $NSG_ID \
      --os-disk-delete-option Delete \
      --data-disk-sizes-gb 30 \
      --storage-sku $STORAGE_SKU \
      --assign-identity \
      --custom-data "install-${i}.sh" \
      --ssh-key-values .ssh/id_rsa.pub \
      --location $LOCATION \
      --public-ip-address "" \
      --tags Env=DEV Department=IT
done
```
## 3.4 Obtener IPs de VMs
```
VM1_IP=$(az vm nic show \
   --nic $PREFIX-vm1VMNic \
   --resource-group "$RG_NAME" \
   --vm-name $PREFIX-vm1 \
   --query "ipConfigurations[0].privateIPAddress" \
   -o tsv)

VM2_IP=$(az vm nic show \
   --nic $PREFIX-vm2VMNic \
   --resource-group "$RG_NAME" \
   --vm-name $PREFIX-vm2 \
   --query "ipConfigurations[0].privateIPAddress" \
   -o tsv)
```
# 4 Crear Application Gateway
## 4.1 Crear IP Publica
```
az network public-ip create \
  --resource-group "$RG_NAME" \
  --name $IP_AGW_NAME \
  --location "$LOCATION" \
  --allocation-method Static \
  --sku Standard
```

```
AGW_IP=$(az network public-ip show --resource-group "$RG_NAME" --name "$IP_AGW_NAME" --query "ipAddress" -o tsv)
```
## 4.2 Crear fichero gateway.json
```
cat << EOF > agw-template.json
{
  "\$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
    {
      "type": "Microsoft.Network/applicationGateways",
      "apiVersion": "2023-11-01",
      "name": "$APGW_NAME",
      "location": "$LOCATION",
      "properties": {
        "sku": {
          "name": "Basic",
          "tier": "Basic",
          "capacity": 1
        },
        "gatewayIPConfigurations": [
          {
            "name": "appGatewayIpConfig",
            "properties": {
              "subnet": {
                "id": "[concat(resourceGroup().id, '/providers/Microsoft.Network/virtualNetworks/', '$VNET_VM_NAME', '/subnets/', '$SUBNET_AGW_NAME')]"
              }
            }
          }
        ],
        "frontendIPConfigurations": [
          {
            "name": "$IP_AGW_NAME",
            "properties": {
              "publicIPAddress": {
                "id": "[concat(resourceGroup().id, '/providers/Microsoft.Network/publicIPAddresses/', '$IP_AGW_NAME')]"
              }
            }
          }
        ],
        "frontendPorts": [
          {
            "name": "$PORT_AGW_NAME",
            "properties": {
              "port": 80
            }
          }
        ],
        "backendAddressPools": [
          {
            "name": "$ADDRESS_POOL_NAME",
            "properties": {
              "backendAddresses": [
                {
                  "ipAddress": "$VM1_IP"
                },
                {
                  "ipAddress": "$VM2_IP"
                }
              ]
            }
          }
        ],
        "backendHttpSettingsCollection": [
          {
            "name": "appGatewayBackendHttpSettings",
            "properties": {
              "port": 80,
              "protocol": "Http",
              "cookieBasedAffinity": "Disabled"
            }
          }
        ],
        "httpListeners": [
          {
            "name": "appGatewayHttpListener",
            "properties": {
              "frontendIPConfiguration": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$APGW_NAME'), '/frontendIPConfigurations/', '$IP_AGW_NAME')]"
              },
              "frontendPort": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$APGW_NAME'), '/frontendPorts/', '$PORT_AGW_NAME')]"
              },
              "protocol": "Http"
            }
          }
        ],
        "requestRoutingRules": [
          {
            "name": "$RULE_AGW_NAME",
            "properties": {
              "ruleType": "Basic",
              "priority": 100,
              "httpListener": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$APGW_NAME'), '/httpListeners/appGatewayHttpListener')]"
              },
              "backendAddressPool": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$APGW_NAME'), '/backendAddressPools/', '$ADDRESS_POOL_NAME')]"
              },
              "backendHttpSettings": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$APGW_NAME'), '/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
              }
            }
          }
        ]
      }
    }
  ]
}
EOF
```
## 4.3 Crear Application Gateway
```
az deployment group create \
  --resource-group "$RG_NAME" \
  --template-file agw-template.json
```
# 5 Verificacion
Abrimos un navegador y colocamos la IP Publica que se creo para Application Gateway.

Desde la terminal ejecutamos:
```
curl -i $AGW_IP
```