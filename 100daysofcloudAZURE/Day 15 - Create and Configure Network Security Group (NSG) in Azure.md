```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a network security group (NSG) with the following requirements:

Name of the NSG should be xfusion-nsg.

Add an inbound security rule named Allow-HTTP for HTTP service on port 80, with the source CIDR range of 0.0.0.0/0.

Add another inbound security rule named Allow-SSH for SSH service on port 22, with the source CIDR range of 0.0.0.0/0.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	
Start Time	Fri Jan 30 18:34:58 UTC 2026
End Time	Fri Jan 30 19:34:58 UTC 2026
```

Listar **Resource Groups**:
```
az group list
```

Capturar _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Crear **Network Security Group** (NSG):
```
az network nsg create \
   --name xfusion-nsg \
   --resource-group $RG_NAME 
```

Crear **security rule**:
```
az network nsg rule create \
--name 'Allow-HTTP' \
--nsg-name xfusion-nsg \
--priority 100 \
--resource-group $RG_NAME \
--direction Inbound \
--access Allow \
--protocol Tcp \
--source-address-prefixes '0.0.0.0/0' \
--source-port-ranges '*' \
--destination-port-ranges 80 \
--description "Permitir trafico http entrante por el puerto 80"
```

```
az network nsg rule create \
--name 'Allow-SSH' \
--nsg-name xfusion-nsg \
--priority 101 \
--resource-group $RG_NAME \
--direction Inbound \
--access Allow \
--protocol Tcp \
--source-address-prefixes '0.0.0.0/0' \
--source-port-ranges '*' \
--destination-port-ranges 22 \
--description "Permitir conexion ssh por el puerto 22"
```

Verificar que se agregaron las reglas:
```
az network nsg show -g $RG_NAME -n xfusion-nsg
```
