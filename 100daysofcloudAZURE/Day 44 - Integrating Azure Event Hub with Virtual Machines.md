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
# Variables de entorno
```
NAMESPACE_NAME=datacenter-namespace
LOCATION=eastus
SKU=Standard
EVENT_HUB_NAME=datacenter-hub
VM_NAME=datacenter-vm
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
   --query [0].name \
   --output tsv)
```
# 1. Crear Azure Event Hubs Namespace
```
az eventhubs namespace create \
   --name $NAMESPACE_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku $SKU \
   --enable-auto-inflate true \
   --maximum-throughput-units 5
```
# 2. Crear Event Hub
```
az eventhubs eventhub create \
   --name $EVENT_HUB_NAME \
   --namespace-name $NAMESPACE_NAME \
   --resource-group $RG_NAME
```
# 3. Obtener cadena de conexion
```
CONNECTION_STRING=$(az eventhubs namespace authorization-rule keys list \
   --name RootManageSharedAccessKey \
   --namespace-name $NAMESPACE_NAME \
   --resource-group $RG_NAME \
   --query primaryConnectionString \
   -o tsv)
```

```
echo "Tu cadena de conexión es: $CONNECTION_STRING"
```
# Configurar y ejecutar script en VM
#### Ver VM
```
az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "{Nombre:name,IPPrivada:privateIps,IPPublica:publicIps,Usuario:osProfile.adminUsername}" \
   -o table
```
#### Obtener IP Publica
```
IP_PUBLICA=$(az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "publicIps" \
   -o tsv)
```
#### Obtener usuario
```
USER=$(az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "osProfile.adminUsername" \
   -o tsv)
```
## Ingresar a VM
```
ssh $USER@$IP_PUBLICA
```
#### Modificar script
Agregamos los credenciales necesarios.
```
vi /home/azureuser/send_logs.py
```
Abrimos otra terminal para luego volver a ejecutar el siguiente comando. Y nos vamos a **Verificacion**.
#### Ejecutar script varias veces
```
# Esto ejecutará el script 10 veces seguidas
for i in {1..10}; do python3 /home/azureuser/send_logs.py; done
```

```
# Esto ejecutará tu script de logs 50 veces seguidas de forma automatizada
for i in {1..50}; do 
   python3 /home/azureuser/send_logs.py
   echo "Log enviado número: $i"
   sleep 0.5
done
```
# Verificacion
#### Instalar dependencia
```
pip install azure-eventhub
```
#### Crear script
```
cat << 'EOF' > view_events.py
import os
from azure.eventhub import EventHubConsumerClient

connection_str = os.environ.get('CONNECTION_STRING')
eventhub_name = os.environ.get('EVENT_HUB_NAME')

def on_event(partition_context, event):
    print(f"--- Evento Recibido desde Partición: {partition_context.partition_id} ---")
    print(event.body_as_str(encoding='UTF-8'))
    partition_context.update_checkpoint(event)

if __name__ == '__main__':
    client = EventHubConsumerClient.from_connection_string(
        conn_str=connection_str,
        consumer_group="$Default",
        eventhub_name=eventhub_name
    )
    print(f"Escuchando eventos en {eventhub_name}... (Presiona Ctrl+C para salir)")
    with client:
        client.receive(on_event=on_event, starting_position="-1")
EOF
```
#### Agregar variables de entorno
```
export CONNECTION_STRING=$CONNECTION_STRING
export EVENT_HUB_NAME=$EVENT_HUB_NAME
```
#### Ejecutar script
```
python3 view_events.py
```
Y volvemos a la anterior terminal ejecutamos **send_logs.py**.
# Ver metricas
```
az monitor metrics list \
  --resource "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/$RG_NAME/providers/Microsoft.EventHub/namespaces/$NAMESPACE_NAME" \
  --metric "IncomingMessages" \
  --interval "PT1M" \
  --aggregation "Total" \
  --query "value[0].timeseries[0].data[?total != null].{Tiempo:timeStamp, TotalMensajes:total}" \
  -o table
```