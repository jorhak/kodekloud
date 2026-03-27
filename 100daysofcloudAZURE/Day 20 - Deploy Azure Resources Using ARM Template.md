```
You are tasked with modifying an ARM template for deploying a virtual network. The current template is located in the /root/arm-templates directory under the filename vnet-deployment-template.json. You need to make the following changes to the template:

Change the name and displayName tag of the virtual network to arm-vnet-nautilus.

Update the addressPrefixes to 192.168.0.0/16.

Add one more tag named Environment with value KKE-nautilus.

After making these changes, you need to deploy the ARM template using the Azure CLI.

Use the following command to find out the resource group to use:

az group list --query '[].name' --output table | grep 'kml'
```

1. Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query '[].name' --output tsv)
```
2. Verificar **Virtual Network**:
```
az network vnet list --resource-group $RG_NAME --output table
```

3. Desplegar template
   - Modificar fichero _vnet-deployment-template.json_:
```
cd arm-templates
vi vnet-deployment-template.json
```

```
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "name": "arm-vnet-nautilus",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2023-11-01",
            "location": "[resourceGroup().location]",
            "tags": {
                "displayName": "arm-vnet-nautilus",
                "Environment": "KKE-nautilus"
            },
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "192.168.0.0/16"
                    ]
                }
            }
        }
    ],
    "outputs": {
    }
}
```

-  Ejecutar comando
```
az deployment group create \
   --resource-group $RG_NAME \
   --template-file vnet-deployment-template.json
```

4. Eliminar **Virtual Network**:
   No ejecutar estos comandos si no son necesarios
```
VN_NAME=$(az network vnet list \
   --resource-group $RG_NAME \
   --query [].name \
   --output tsv)
   
az network vnet delete \
   --resource-group $RG_NAME \
   --name $VN_NAME
```
