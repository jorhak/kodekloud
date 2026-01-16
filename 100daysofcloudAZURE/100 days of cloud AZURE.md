VAMOS A REALIZAR LOS RETOS DE LA NUVE DE AZURE

# Day 1: Create SSH Key Pair for Azure Virtual Machine
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create an SSH key pair with the following requirements:

The name of the SSH key pair should be devops-kp.

The key pair type must be rsa.



Use below given Azure Credentials: (You can run the showcreds command on the azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main-4107ddd6ee5c4b9c@azurefreekmlprod.onmicrosoft.com
Password	msgSR9YS
Start Time	Fri Jan 16 14:44:00 UTC 2026
End Time	Fri Jan 16 15:44:00 UTC 2026
```

Vamos a comenzar listando resource group para poder las llaves ahi:
```
az group list
```

Vamos a crear el par de llaves:
```
az sshkey create \
  --name "devops-kp" \
  --resource-group "kml_rg_main-4107ddd6ee5c4b9c" \
  --location "westus" \
  --encryption-type RSA \
  --tags "Environment=Desarrollo" "Owner=Mario Parez"
```

Verificar que se creo el par de llaves
```
az sshkey list
```