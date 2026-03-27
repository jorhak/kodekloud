```
The Nautilus DevOps team is strategically planning the migration of a portion of their infrastructure to the Azure cloud. Acknowledging the magnitude of this endeavor, they have chosen to tackle the migration incrementally rather than as a single, massive transition. Their approach involves creating Virtual Networks (VNets) as the initial step, as they will be provisioning various services under different VNets.

Create a Virtual Network (VNet) named `nautilus-vnet` in the `westus` region with `192.168.0.0/24` IPv4 CIDR.  
  

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on the `azure-client` host to retrieve credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Tue Jan 20 20:30:47 UTC 2026|
|End Time|Tue Jan 20 21:30:47 UTC 2026|
```

Lo primero que vamos hacer sera ver **resource group**:
```
az group list
```

Capturamos su _name_:
```
RG_NAME=$(az group list --query "[].name" --output tsv)
REGION=$(az group list --query "[].location" --output tsv)
VNET_NAME=nautilus-vnet
```

Creamos la **VNet**:
```
az network vnet create \
   -g $RG_NAME \
   -n $VNET_NAME \
   --location $REGION \
   --address-prefixes 192.168.0.0/24
```

Listar **VNet**:
```
az network vnet list
```
