```
The Nautilus Development Team needs to set up a new Azure Virtual Machine (VM) and configure it to run a web server. This VM should be part of an Azure Application Gateway (AGW) setup to ensure high availability and better traffic management. The task involves creating a VM, setting up an AGW, configuring a backend pool, and ensuring the web server is accessible via the AGW public IP.

**Create a Network Security Group (NSG):** Create an NSG named `xfusion-nsg` and add an inbound security rule `Allow-HTTP` to allow `TCP` traffic on port `80`.

**Create a Virtual Machine:** Create a VM named `xfusion-vm` using any available Ubuntu image. Configure the instance with the following settings:

- **Size:** Choose a lightweight VM size (e.g., `Standard_B1s`).
    
- **Authentication:** Use `SSH public key` authentication. (Please select `use existing public key` option, create public-key locally and paste contents of `~/.ssh/id_rsa.pub`)
    
- **OS Disk:** Use a `Standard HDD`.
    
- **Networking:** Under the Advanced section, attach an existing NSG (e.g., `xfusion-nsg`).
    

Additionally, configure the instance to run a user data script during launch that:

- Install the Nginx package.
    
- Start the Nginx service.
    

**Set up an Application Gateway:** Set up an Azure Application Gateway named `xfusion-agw` with the following:

- Create and Associate it with a **public IP address** named `xfusion-agw-ip`.
    
- Attach the **backend pool**:`xfusion-backendpool` to the VM `xfusion-vm`.
    
- Select a **subnet** for the Application Gateway (you can create a new one if needed).
    

**Configure HTTP Settings:** Create an HTTP setting named `xfusion-http-settings` on port `80`

**Route Traffic:** Add a **listener** named `xfusion-listener` and a **routing rule** named `xfusion-routing-rule` to route traffic from the AGW frontend to the backend pool:

- Listener: Frontend IP = public IP, Frontend port = 80, Protocol = HTTP
    
- Routing rule: Connects `xfusion-listener` to `xfusion-backendpool` using `xfusion-http-settings`.
    

**NSG Adjustments:** Make sure the **NSG** attached to the VM allows **inbound TCP traffic on port 80**, so the Nginx server running on `xfusion-vm` is accessible via the Application Gateway public IP.

**Note:** Wait for the Application Gateway resource to be fully deployed before proceeding with the next steps. Deployment may take several minutes to complete.

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azure.onmicrosoft.com]|
|Password|contra|
|Start Time|Tue Jun 30 15:14:30 UTC 2026|
|End Time|Tue Jun 30 16:14:30 UTC 2026|

  
`Notes:`

- Create the resources only in the `West US` region.
- To `display` or `hide` the terminal of the Azure client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```
# 1. Variables de entorno
```
NSG_NAME=devops-nsg
RSG_NAME=Allow-HTTP
VM_NAME=devops-vm
VM_SIZE=Standard_B1s
VM_STORAGE=Standard_LRS
VNET_NAME=devops-vnet
SUBNET_NAME=vm-subnet
AGW_NAME=devops-agw
IP_PUBLIC_NAME=devops-agw-ip
BACKEND_POOL_NAME=devops-backendpool
HTTP_SETTINGS_NAME=devops-http-settings
LISTENER_NAME=devops-listener
ROUTING_RULE_NAME=devops-routing-rule

LOCATION=westus
AGW_SUBNET_NAME=agw-subnet
SSH_VALUES=$HOME/.ssh/id_rsa.pub
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
   --query [0].name \
   --output tsv)
```
# Listar todos los recursos de Resource Group
```
az resource list \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --query "[*].{Nombre:name,Tipo:type}" \
   -o table
```
# 2. Crear VNet y Subred para VM
```
az network vnet create \
  --resource-group "$RG_NAME" \
  --name $VNET_NAME \
  --address-prefixes 10.0.0.0/16 \
  --subnet-name $SUBNET_NAME \
  --subnet-prefixes 10.0.1.0/24 \
  --location "$LOCATION"
```
#### Crear la subred dedicada para el Application Gateway
```
az network vnet subnet create \
  --resource-group "$RG_NAME" \
  --vnet-name $VNET_NAME \
  --name $AGW_SUBNET_NAME \
  --address-prefixes 10.0.2.0/24
```
# 3. Crear Network Security Group
```
az network nsg create \
   --name $NSG_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION
```
#### Agregar regla
```
az network nsg rule create \
   --resource-group $RG_NAME \
   --nsg-name $NSG_NAME \
   --name $RSG_NAME \
   --protocol Tcp \
   --direction Inbound \
   --priority 100 \
   --access Allow \
   --source-address-prefixes "*" \
   --source-port-ranges "*" \
   --destination-address-prefixes "*" \
   --destination-port-ranges 80 \
   --description "Permitir trafico por el puerto 80 http"
```
# 4. Crear VM
#### User data
```
vi install.sh
```

```
#!/bin/bash
sudo apt-get update
sudo apt-get install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```
#### Crear llaves
```
ssh-keygen -t rsa -b 4096
```
#### Crear VM
```
az vm create \
   --resource-group $RG_NAME \
   --name $VM_NAME \
   --location $LOCATION \
   --image Ubuntu2204 \
   --admin-username "azureuser" \
   --custom-data install.sh \
   --nsg $NSG_NAME \
   --size $VM_SIZE \
   --storage-sku $VM_STORAGE \
   --ssh-key-values "$SSH_VALUES" \
   --vnet-name $VNET_NAME \
   --subnet $SUBNET_NAME 
```
Si agregamos el flag **--public-ip-address ""** esta nos va crear la **VM** solamente con **IP Privada**, sino la agregamos esta crea la **VM** con **IP Publica y Privada**.

```
az vm show -d --resource-group "$RG_NAME" --name "$VM_NAME" --query "{Nombre:name,IPPrivada:privateIps,IPPublica:publicIps,Usuario:osProfile.adminUsername}" -o table
```
#### Obtener IP Privada
```
VM_PRIVATE_IP=$(az vm show -d --resource-group "$RG_NAME" --name "$VM_NAME" --query "privateIps" -o tsv)
```
# 5. Crear IP Publica
```
az network public-ip create \
   --name $IP_PUBLIC_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --allocation-method Static \
   --sku Standard
```
#### Obtener IP Public
```
IP_PUBLIC_AGW=$(az network public-ip show \
   --name $IP_PUBLIC_NAME \
   --resource-group $RG_NAME \
   --query "ipAddress" \
   -o tsv)
```
# 6. Generar plantilla (Bypass de la Policy)
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
      "name": "$AGW_NAME",
      "location": "$LOCATION",
      "properties": {
        "sku": {
          "name": "Basic",
          "tier": "Basic",
          "capacity": 2
        },
        "gatewayIPConfigurations": [
          {
            "name": "appGatewayIpConfig",
            "properties": {
              "subnet": {
                "id": "[concat(resourceGroup().id, '/providers/Microsoft.Network/virtualNetworks/', '$VNET_NAME', '/subnets/', '$AGW_SUBNET_NAME')]"
              }
            }
          }
        ],
        "frontendIPConfigurations": [
          {
            "name": "appGatewayFrontendIP",
            "properties": {
              "publicIPAddress": {
                "id": "[concat(resourceGroup().id, '/providers/Microsoft.Network/publicIPAddresses/', '$IP_PUBLIC_NAME')]"
              }
            }
          }
        ],
        "frontendPorts": [
          {
            "name": "appGatewayFrontendPort",
            "properties": {
              "port": 80
            }
          }
        ],
        "backendAddressPools": [
          {
            "name": "$BACKEND_POOL_NAME",
            "properties": {
              "backendAddresses": [
                {
                  "ipAddress": "$VM_PRIVATE_IP"
                }
              ]
            }
          }
        ],
        "backendHttpSettingsCollection": [
          {
            "name": "$HTTP_SETTINGS_NAME",
            "properties": {
              "port": 80,
              "protocol": "Http",
              "cookieBasedAffinity": "Disabled"
            }
          }
        ],
        "httpListeners": [
          {
            "name": "$LISTENER_NAME",
            "properties": {
              "frontendIPConfiguration": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$AGW_NAME'), '/frontendIPConfigurations/appGatewayFrontendIP')]"
              },
              "frontendPort": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$AGW_NAME'), '/frontendPorts/appGatewayFrontendPort')]"
              },
              "protocol": "Http"
            }
          }
        ],
        "requestRoutingRules": [
          {
            "name": "$ROUTING_RULE_NAME",
            "properties": {
              "ruleType": "Basic",
              "priority": 100,
              "httpListener": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$AGW_NAME'), '/httpListeners/$LISTENER_NAME')]"
              },
              "backendAddressPool": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$AGW_NAME'), '/backendAddressPools/$BACKEND_POOL_NAME')]"
              },
              "backendHttpSettings": {
                "id": "[concat(resourceId('Microsoft.Network/applicationGateways', '$AGW_NAME'), '/backendHttpSettingsCollection/$HTTP_SETTINGS_NAME')]"
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
#### Ejecutar despliegue del grupo
```
az deployment group create \
  --resource-group "$RG_NAME" \
  --template-file agw-template.json
```
# 7. Verificacion
#### Obtener IP Privada
```
AGW_IP=$(az network public-ip show --resource-group "$RG_NAME" --name "$IP_PUBLIC_NAME" --query "ipAddress" -o tsv)
```
#### Realizar peticion
```
curl -I http://$AGW_IP
```

# Eliminar todos los recursos
```
for id in $(az resource list --resource-group $RG_NAME --query "[].id" -o tsv); do
    az resource delete --ids "$id" --verbose
done

```