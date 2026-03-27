```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

Create a managed disk with the following requirements:

Name of the disk should be datacenter-disk.

Disk type must be Standard_LRS.

Disk size must be 2 GiB.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Fri Jan 30 18:10:25 UTC 2026
End Time	Fri Jan 30 19:10:25 UTC 2026
```

Listar **Resource Groups**:
```
az group list
```

Capturar _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Crear **disk**:
```
az disk create -g $RG_NAME -n datacenter-disk --sku Standard_LRS --size-gb 2
```

Listar **disk**:
```
az disk list -g $RG_NAME
```
