```
The Nautilus DevOps team is currently working on setting up a simple application on the Azure cloud. They aim to establish an Azure Load Balancer in front of a Virtual Machine (VM) where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later.

Set up an Azure Load Balancer named devops-lb.
Configure the Load Balancer’s frontend IP configuration with the name devops-lb-ip and assign a public IP address with the same name (devops-lb-ip).
Create a backend pool named devops-backend-pool and add the VM running Nginx to this pool.
Create a health probe named devops-health-probe on port 80 to check the VM's health.
Set up a load balancer rule named devops-lb-rule to route traffic on port 80 to the backend pool on port 80.
Add an inbound rule to the existing NSG of the VM to allow HTTP traffic on port 80.




Use the Azure Portal URL and login credentials below:

Console URL	https://portal.azure.com/azurefreekmlprod.onmicrosoft.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Thu May 14 14:05:21 UTC 2026
End Time	Thu May 14 15:05:21 UTC 2026

Notes:

Create the resources only in the eastus region.

To display or hide the terminal on the Azure client machine, you can use the expand toggle button.
```
# Variables de entorno
```
LB_NAME=nautilus-lb
LB_IP_NAME=nautilus-lb-ip
BACKEND_POOL_NAME=nautilus-backend-pool
HEALTH_NAME=nautilus-health-probe
PORT=80
RULE_NAME=nautilus-lb-rule
LOCATION=eastus
```
# Capturar el Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# Ver todos los recursos de Resource Group
Ya que nos indican que debemos agregar una VM a un Backend Pool necesito conocer que objetos tengo en mi Resource Group:
```
az resource list \
   -g $RG_NAME \
   --query "[*].{NOMBRE:name,TIPO:type}" \
   --output table
```
# Capturar Name de VM
Capturamos el nombre de VM para obtener otros objetos que seran necesarios:
```
VM_NAME=$(az vm list \
   -g $RG_NAME \
   --query "[0].name" \
   --output tsv)
```
### Ver detalles de VM
```
az vm show \
   -n $VM_NAME \
   -g $RG_NAME \
   -d \
   --query "{NOMBRE:name, USERNAME:osProfile.adminUsername, ESTADO:powerState, IpPRIVADA:privateIps, IpPUBLICA:publicIps}" \
   --output table
```

### Capturar Network Security Group ID
#### Primero obtenemos el NIC_ID y NIC_NAME
Antes de capturar NGS ID debemos obtener el Network Interfaces Card (NIC_ID, NIC_NAME) el cual lo vamos a obtner de los atributos de VM:
```
NIC_ID=$(az vm show \
   -n $VM_NAME \
   -g $RG_NAME \
   -d \
   --query "networkProfile.networkInterfaces[0].id" \
   --output tsv)
```

```
NIC_NAME=$(az network nic show \
   --ids $NIC_ID \
   --query "name" \
   --output tsv)
```
#### Segundo obtenemos el NSG_ID y NSG_NAME
Cuando ya tenemos NIC_ID recien podemos tener NSG_ID y NSG_NAME el cual esta relacionado a nuestra VM:
```
NSG_ID=$(az network nic show \
   --ids $NIC_ID \
   --query "networkSecurityGroup.id" \
   --output tsv)
```

```
NSG_NAME=$(az network nsg show \
   --ids $NSG_ID \
   --query "name" \
   --output tsv)
```
# 1 Puerta de entrada y balanceador
### Crear IP Publica
Creamos una IP Publica para asignarla a nuestro LB:
```
az network public-ip create \
   --name $LB_IP_NAME \
   --resource-group $RG_NAME \
   --sku Standard \
   --location $LOCATION
```
### Crear Load Balancer
```
az network lb create \
   --name $LB_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku Standard \
   --frontend-ip-name $LB_IP_NAME \
   --public-ip-address $LB_IP_NAME
```
# 2 Backend
### Crear pool
Este Pool es el encargado de recibir nuestra carga de peticiones:
```
az network lb address-pool create \
   --name $BACKEND_POOL_NAME \
   --lb-name $LB_NAME \
   --resource-group $RG_NAME
```
### Health
Se va a encargar de ver el estado de nuestro servico que esta corriendo.
```
az network lb probe create \
   --lb-name $LB_NAME \
   --name $HEALTH_NAME \
   --port $PORT \
   --protocol http \
   --resource-group $RG_NAME \
   --path "/"
```
### Rule
Creamos una regla en donde debemos colocar el HEALTH previamente creado para asignarlo al LB. Cada cierto tiempo se enviaran peticiones para ver el estado del servicio:
```
az network lb rule create \
   --backend-port $PORT \
   --frontend-port $PORT \
   --lb-name $LB_NAME \
   --name $RULE_NAME \
   --protocol tcp \
   --resource-group $RG_NAME \
   --backend-pool-name $BACKEND_POOL_NAME \
   --frontend-ip-name $LB_IP_NAME \
   --probe-name $HEALTH_NAME
```
# 3 Open Port
Abrir puerto de la VM, para aceptar peticiones:
```
az network nsg rule create \
  --name "AllowHTTPLB" \
  --nsg-name $NSG_NAME \
  --priority 100 \
  --resource-group $RG_NAME \
  --direction Inbound \
  --access Allow \
  --protocol tcp \
  --destination-port-ranges $PORT \
  --description "Permitir trafico para LB"
```
# 4 Asociar NIC al Backend Pool del Load Balancer
Como vimos previamente se creo un Backend Pool, sin embargo, este no cuenta con los recursos para recibir la carga de peticiones es por eso que asociamos nuestra VM a traves de NIC para recibir el trafico:
#### Obtener IP CONFIG NAME
```
IP_CONFIG_NAME=$(az network nic ip-config list \
    --resource-group $RG_NAME \
    --nic-name $NIC_NAME \
    --query "[0].name" \
    -o tsv)
```
#### Asociar NIC con Backend Pool
```
az network nic ip-config address-pool add \
   --address-pool $BACKEND_POOL_NAME \
   --ip-config-name $IP_CONFIG_NAME \
   --nic-name $NIC_NAME \
   --resource-group $RG_NAME \
   --lb-name $LB_NAME
```
# Verificar
### Obtener ip de Load Balancer
```
az network public-ip show \
   --resource-group $RG_NAME \
   --name $LB_IP_NAME \
   --query "ipAddress" \
   --output tsv
```
Abrimos un navegador y colocamos
```
http://<IP LOAD BALANCER>
```
Nos tiene que mostrar la pagina de Nginx.

```
curl -I --connect-timeout 5 http://<IP LOAD BALANCER>
```

```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 16 May 2026 14:58:24 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Sat, 16 May 2026 14:52:10 GMT
Connection: keep-alive
ETag: "6a08849a-264"
Accept-Ranges: bytes
```

### Ingresar por SSH (OPCIONAL)
```
IP_PUBLIC=$(az vm show \
   -n $VM_NAME \
   -g $RG_NAME \
   -d \
   --query publicIps \
   --output tsv)
USER=azureuser
```

```
ssh $USER@$IP_PUBLIC
```
### Vamos a detener Nginx
Abrimos otra terminal y vamos a observar los logs:
```
tail -f /var/log/nginx/access.log
```
Esto para ver que nos muestra LB.
```
sudo systemctl stop nginx
```

