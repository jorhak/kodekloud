```
The Nautilus DevOps team wants to integrate an Azure Virtual Machine with Azure Event Hubs and Azure Blob Storage for centralized log collection and backup. Follow these steps to complete the task:

Create Azure Event Hubs Namespace:

Create an Event Hubs namespace named datacenter-namespace in the East US region.
Select the Standard pricing tier. Make sure to enable Enable Auto-inflate.

Create an Event Hub:
Within the namespace, create an Event Hub named datacenter-hub.

Set Up Azure Blob Storage for Log Backup:
Create a Storage Account named datacenterst3336 in the East US region.
Create a container named datacenter-backup-20468 within the Storage Account.
Ensure the container is publicly accessible for read operations.

Verify the Virtual Machine Configuration:
The client host already has a Python script named send_logs.py located under /root. This script is used to send logs to the Event Hub.

Create a Virtual Machine named datacenter-vm in the East US region.

Copy the send_logs.py script from the client host to the /home/azureuser directory of the datacenter-vm.

Modify the script on the VM to also back up the logs to the datacenter-backup-20468 container in the Azure Blob Storage account.

Verify Logs:

Ensure the logs are successfully sent to the Event Hub by checking the Event Hubs metrics in the Azure portal.

Verify that the logs are backed up to the datacenter-backup-20468 container in the Azure Blob Storage.

Use the Azure portal URL and login credentials below:

Console URL https://portal.azure.com/azurefreekmlprod.onmicrosoft.com
Username kk_lab_user_main@azure.onmicrosoft.com
Password contra
Start Time Fri Jul 03 15:38:57 UTC 2026
End Time Fri Jul 03 16:38:57 UTC 2026
Notes:

Create the resources only in the East US region.

Use the existing client host to copy the script to the VM.

Verify both the Event Hubs metrics and the Blob Storage container for successful log ingestion and backup.
```
# Variables de entorno
```
NAMESPACE_NAME=xfusion-namespace
LOCATION=eastus
NAMESPACE_SKU=Standard
EVENT_HUB_NAME=xfusion-hub
SA_NAME=xfusionst29609
SA_SKU=Standard_LRS
CONTAINER_NAME=xfusion-backup-2720
VM_NAME=xfusion-vm
SRC_FILE=/root/send_logs.py
DEST_FILE=/home/azureuser
VM_SIZE=Standard_B1s
VM_SKU=Standard_LRS
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
    --query "[0].name" \
    --output tsv)
```
# Crear Azure Event Hubs Namespace
```
az eventhubs namespace create \
   --name $NAMESPACE_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku $NAMESPACE_SKU \
   --enable-auto-inflate true \
   --maximum-throughput-units 5
```
# Crear Event Hub
```
az eventhubs eventhub create \
   --name $EVENT_HUB_NAME \
   --namespace-name $NAMESPACE_NAME \
   --resource-group $RG_NAME
```
# Crear cadena de conexion
```
CONNECTION_STRING=$(az eventhubs namespace authorization-rule keys list \
   --name RootManageSharedAccessKey \
   --namespace-name $NAMESPACE_NAME \
   --resource-group $RG_NAME \
   --query primaryConnectionString \
   -o tsv)
```

```
echo "Cadena de conexión de EventHubs Namespace::: $CONNECTION_STRING"
```
# Crear Storage Account
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku $SA_SKU \
   --allow-blob-public-access true
```
# Obtener Connection String de Storage Account
```
STORAGE_CONN=$(az storage account show-connection-string --resource-group "$RG_NAME" --name "$SA_NAME" --query connectionString -o tsv)
```

```
echo "Cadena de conexión de Storage Account::: $STORAGE_CONN"
```
# Crear Container
```
az storage container create \
   --name $CONTAINER_NAME \
   --connection-string $STORAGE_CONN \
   --public-access container
```
# Crear VM
```
ssh-keygen -t rsa -b 4096
```

```
az vm create \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --image $IMAGE \
   --location $LOCATION \
   --admin-username azureuser \
   --size $VM_SIZE \
   --storage-sku $VM_SKU \
   --ssh-key-values /root/.ssh/id_rsa.pub
```
# Mover fichero
#### Ver VM
```
az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "{Nombre:name,IPPrivada:privateIps,IPPublica:publicIps,Usuario:osProfile.adminUsername}" \
   -o table
```
#### Capturar IP publica de VM
```
IP_PUBLICA=$(az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "publicIps" \
   -o tsv)
```
#### Capturar usuario de VM
```
USER=$(az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "osProfile.adminUsername" \
   -o tsv)
```
#### Copiar fichero del host al servidor
```
scp $SRC_FILE $USER@$IP_PUBLICA:$DEST_FILE/
```
# Ingresar a VM y modificar fichero
```
ssh $USER@$IP_PUBLICA
```
#### Instalar paquetes y verificar que se instalaron correctamente
```
sudo apt update
sudo apt install python3-pip python3-venv -y
python3 -m venv azure-env
source azure-env/bin/activate
pip install azure-eventhub azure-storage-blob
python3 -c 'import azure.eventhub, azure.storage.blob; print("Librerias cargadas correctamente")'
```
#### Modificar fichero
Agregamos CONNECTION_STRING y STORAGE_CONN
```
vi /home/azureuser/send_logs.py
```
Abrimos otra terminal y capturamos los valores ya mencionados
```
import os
from azure.storage.blob import BlobServiceClient
from azure.eventhub import EventHubProducerClient, EventData

# Event Hub Configuration
eventhub_conn_str = "<Event Hub Connection String>"
eventhub_name = "datacenter-hub"
producer = EventHubProducerClient.from_connection_string(eventhub_conn_str, eventhub_name=eventhub_name)

# Blob Storage Configuration
blob_conn_str = "<Blob Storage Connection String>"
blob_service_client = BlobServiceClient.from_connection_string(blob_conn_str)
blob_client = blob_service_client.get_blob_client(container="datacenter-backup-24900", blob="logs.txt")

# Generate and Send Logs
log_data = "Log entry from VM\n"

# Send to Event Hub
event_data_batch = producer.create_batch()
event_data_batch.add(EventData(log_data))
producer.send_batch(event_data_batch)

# Backup to Blob Storage
blob_client.upload_blob(log_data, blob_type="AppendBlob", overwrite=True)
print("Log sent to Event Hub and backed up to Blob Storage.")
```
# Verificacion
## Instalar dependencias
Abrir otra terminal
```
pip install azure-eventhub
```
## Crear Script
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
## Agregar variables de entorno
```
export CONNECTION_STRING=$CONNECTION_STRING
export EVENT_HUB_NAME=$EVENT_HUB_NAME
```
## Ejecutar Script
```
python3 view_events.py
```
## Volvemos al servidor y ejecutamos el script
```
python3 send_logs.py
```
## Verificar que los ficheros suben a Container
Abrimos otra terminal
```
az storage blob list \
  --container-name "$CONTAINER_NAME" \
  --connection-string "$STORAGE_CONN" \
  --prefix logs
```

Abrimos el navegador y colocamos la url, nombre del container, nombre del fichero
### https://xfusionst29609.blob.core.windows.net/xfusion-backup-2720/logs.txt

Otra forma es descargar todos los ficheros del contenedor para luego ver su contenido
```
az storage blob download-batch \
   --destination /opt \
   --source $CONTAINER_NAME \
   --connection-string $STORAGE_CONN
```

```
cd /opt
cat logs.txt
```