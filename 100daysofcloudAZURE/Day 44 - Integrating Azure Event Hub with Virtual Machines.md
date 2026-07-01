```
The Nautilus DevOps team wants to integrate an Azure Virtual Machine with Azure Event Hubs for centralized log collection. Follow these steps to complete the task:

Create Azure Event Hubs Namespace:

Create an Event Hubs namespace named devops-namespace in the East US region.
Select the Standard pricing tier. Make sure to enable Enable Auto-inflate.
Create an Event Hub:

Within the namespace, create an Event Hub named devops-hub.
Verify the Virtual Machine Configuration:

A VM named devops-vm already exists.
A Python script named send_logs.py already exists on the VM under /home/azureuser. This script is used to send logs to the Event Hub. Make sure to execute this script mutiple times.
Verify Logs:

Ensure the logs are successfully sent to the Event Hub by checking the Event Hubs metrics in the Azure portal.

Use the Azure portal URL and login credentials below:

Console URL	https://portal.azure.com/azurefreekmlprod.onmicrosoft.com
Username	kk_lab_user_main@azure.onmicrosoft.com
Password	contra
Start Time	Wed Jul 01 01:12:20 UTC 2026
End Time	Wed Jul 01 02:12:20 UTC 2026
Notes:

Create the resources only in the East US region.
Use the existing VM devops-vm to send logs.
Verify the Event Hubs metrics to confirm successful log ingestion.
```
# 1. Variables de entorno
```
NAMESPACE_NAME=devops-namespace
location=eastus
SKU=Standard
EVENT_HUB_NAME=devops-hub
VM_NAME=devops-vm
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
   --query [0].name \
   --output tsv)
```
# Crear Azure Event Hubs Namespace
```
az eventhubs namespace create \
   --name \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku $SKU
```
# Crear Event Hub
```
az eventhubs eventhub create \
   --name \
   --namespace-name \
   --resource-group
```
# Verificar VM
```
az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "{Nombre:name,IPPrivada:privateIps,IPPublica:publicIps,Usuario:osProfile.adminUsername}" \
   -o table
```