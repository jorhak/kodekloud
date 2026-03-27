```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations.

Create a Virtual Network (VNet) named xfusion-vnet in the westus region with any IPv4 CIDR block.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Tue Jan 20 17:58:50 UTC 2026
End Time	Tue Jan 20 18:58:50 UTC 2026
```

Lo primero que vamos hacer sera ver si hay algun **resource group**:
```
az group list
```

En nuestro caso existe un **resource group** vamos a capturar su _name y location_:
```
RG_NAME=$(az group list --query "[].{name:name}" --output tsv)
REGION=$(az group list --query "[].{location:location}" --output tsv)
VNET_NAME=xfusion-vnet
```

Crear **Virtual Network**:
```
az network vnet create \
   -g $RG_NAME \
   -n $VNET_NAME \
   --address-prefixes 50.0.0.0/24
```

Listar **Virtual Network**:
```
az network vnet list
```
