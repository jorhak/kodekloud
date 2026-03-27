```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, allocate a Public IP address, name it as datacenter-pip.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Thu Jan 22 14:29:40 UTC 2026
End Time	Thu Jan 22 15:29:40 UTC 2026
```

Lo primero que debemos hacer es ver **Resource Group**:
```
az group list
```

Capturamos _name_:
```
RG_NAME=$(az group list --query "[0].name" --output tsv)
```

Crear **IP Publica**:
```
IP_PUBLIC_NAME=datacenter-pip
az network public-ip create \
   --name $IP_PUBLIC_NAME \
   --resource-group $RG_NAME \
   --version IPv4
```

Verificar la creacion de la **IP Publica**:
```
az network public-ip list -g $RG_NAME
```
