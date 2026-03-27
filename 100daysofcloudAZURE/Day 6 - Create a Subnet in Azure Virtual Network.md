```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition.

For this task, create a Virtual Network (VNet) named `datacenter-vnet` and one subnet named `datacenter-subnet` within the VNet in the `centralus` region. Make sure the `IPv4 address range` is `10.0.0.0/16`.  
  

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azurefree.onmicrosoft.com]|
|Password|contra|
|Start Time|Wed Jan 21 20:30:52 UTC 2026|
|End Time|Wed Jan 21 21:30:52 UTC 2026|
```

Crear variables:
```
VNET_NAME=datacenter-vnet
SNET_NAME=datacenter-subnet
REGION=centralus
```

Listar **Resource Group**:
```
az group list
```

Capturar _name_ del **Resource Group**:
```
RESOURCE_GROUP_NAME=$(az group list --query "[0].name" --output tsv)
```

Ahora vamos a comenzar con la creacion de **VNet**:
```
az network vnet create \
   --name $VNET_NAME \
   --resource-group $RESOURCE_GROUP_NAME \
   --location $REGION \
   --address-prefixes 10.0.0.0/16
   
```

Seguimos con la **Subnet**:
```
az network vnet subnet create \
   --name $SNET_NAME \
   --resource-group $RESOURCE_GROUP_NAME \
   --vnet-name $VNET_NAME \
   --address-prefixes 10.0.0.0/16
```
