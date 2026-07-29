```
The Nautilus DevOps team needs to set up an Azure Virtual Machine (VM) to interact with an Azure Blob Storage container for storing and retrieving data. The team must create a private storage account, configure Blob Storage, and test the functionality.

Task:
1) Azure Virtual Machine Setup:
Create a VM named nautilus-vm in the East US region.
Authentication: Use SSH public key authentication. (Please select use existing public key option, create public-key locally and paste contents of ~/.ssh/id_rsa.pub).
Install Docker and Azure CLI on the VM.
Pull the Docker image from the ACR and run it on the VM, ensuring it listens on port 80.

2) Azure Container Registry (ACR) Setup:
Create an ACR named nautilusacr2520 in the East US region.
The repository name should be nautilus/python-app.
Build the Docker image using the Dockerfile already given under /root/pyapp.
Push the Docker image to the ACR with the tag latest.

3) Create a Storage Account and Blob Container:
Create a storage account named nautilusstor2520 in the East US region with Locally-redundant storage (LRS).
Create a Blob container named nautilus-config.
Upload a file named config.json (available under /root/) to the container.

4) Validation:
Confirm that the application can be accessed on the browser.

Use below given Azure Portal Credentials:

Portal URL https://portal.azure.com
Username kk_lab_user_main@azure.onmicrosoft.com
Password contra
Start Time Tue Jul 07 01:33:32 UTC 2026
End Time Tue Jul 07 02:33:32 UTC 2026
Notes:

Create all resources in the East US region.
Use the Azure CLI or Azure Portal for resource creation.
The Dockerfile is already given under /root/pyapp. The image tag must be latest.
The repository name should be nautilus/python-app.
```
# Variables de entorno
```
PREFIX=datacenter
VM_NAME=$PREFIX-vm
IMAGE=Ubuntu2404
LOCATION=eastus
NSG_NAME=$PREFIX-nsg
RSG_NAME_SSH=AllowSSH
RSG_NAME_HTTP=AllowHTTP
SIZE=Standard_B1s
STORAGE_SKU=Standard_LRS
ACR_NAME="${PREFIX}acr10389"
ACR_SKU=Basic
REPO_NAME=$PREFIX/python-app
SA_NAME="${PREFIX}stor10389"
BC_NAME=$PREFIX-config
UPLOAD_FILE=/root/config.json
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
     --query [0].name \
     --output tsv)
```
# 1 Crear Security Group
```
NSG_ID=$(az network nsg create \
  --name $NSG_NAME \
  --resource-group $RG_NAME \
  --location $LOCATION \
  --query NewNSG.id \
  --output tsv)
```
Otra forma de obtener el ID
```
NSG_ID=$(az network nsg show \
  --name $NSG_NAME \
  --resource-group $RG_NAME \
  --query id  \
  --output tsv)
```
## 1.1 Agregamos la regla para SSH
```
az network nsg rule create \
  --nsg-name $NSG_NAME \
  --resource-group $RG_NAME \
  --name $RSG_NAME_SSH \
  --protocol tcp \
  --direction Inbound \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 22 \
  --access Allow \
  --priority 100
```
## 1.2 Agregamos la regla para HTTP
```
az network nsg rule create \
  --nsg-name $NSG_NAME \
  --resource-group $RG_NAME \
  --name $RSG_NAME_HTTP \
  --protocol tcp \
  --direction Inbound \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 80 \
  --access Allow \
  --priority 101
```
# 2 Crear VM
## 2.1 Crear par de llaves
```
ssh-keygen -t rsa -b 4096
```
## 2.2 Script de instalacion
```
vi install.sh
```

```
#!/bin/bash
#Filename: install.sh
#Description: Instalar Docker y Azure cli

export DEBIAN_FRONTEND=noninteractive
export NEEDRESTART_MODE=a
USUARIO=azureuser

sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)

# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt-get update
sudo -E apt-get install -y \
  -o Dpkg::Options::="--force-confdef" \
  -o Dpkg::Options::="--force-confold" \
  docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable docker
sudo systemctl start docker
sudo groupadd docker 2>/dev/null || true
sudo usermod -aG docker $USUARIO
newgrp docker
sudo chmod 660 /var/run/docker.sock
sudo chown root:docker /var/run/docker.sock

curl -fsSL 'https://azurecliprod.blob.core.windows.net/$root/deb_install.sh' | sudo bash
```
## 2.3 Crear VM
```
az vm create \
    --resource-group $RG_NAME \
    --name $VM_NAME \
    --image $IMAGE \
    --size $SIZE \
    --nsg $NSG_ID \
    --data-disk-sizes-gb 30 \
    --storage-sku $STORAGE_SKU \
    --assign-identity \
    --custom-data install.sh \
    --ssh-key-values .ssh/id_rsa.pub
```

```
az vm wait -g $RG_NAME -n $VM_NAME --created
```
# 3 Crear ACR
```
az acr create \
   --name $ACR_NAME \
   --resource-group $RG_NAME \
   --sku $ACR_SKU \
   --location $LOCATION
```

```
az acr update --name "$ACR_NAME" --admin-enabled true
```
# 4 Crear imagen y subirla al repositorio
## 4.1 Modificar Dockerfile y mover fichero config.json
```
vi pyapp/Dockerfile
```

```
COPY config.json /app/
```
Guardamos los cambios
```
cp config.json /root/pyapp/
```
Copiamos el fichero config.json
## 4.2 Iniciar sesion localmente
```
az acr login --name "$ACR_NAME"
```
## 4.3 Crear imagen localmente
```
docker build -t "${ACR_NAME}.azurecr.io/$REPO_NAME:latest" /root/pyapp/
```
## 4.4 Subir imagen al repositorio
```
docker push "${ACR_NAME}.azurecr.io/$REPO_NAME:latest"
```
# 5 Verificar que la imagen subio al repositorio
```
az acr repository list \
   -n $ACR_NAME \
   --output tsv
```

```
az acr repository show \
   --name $ACR_NAME \
   --repository $REPO_NAME
```
# 6 Crear Storage Account
```
az storage account create \
   --name $SA_NAME \
   --resource-group $RG_NAME \
   --location $LOCATION \
   --sku Standard_LRS \
   --allow-blob-public-access true \
   --public-network-access Enabled
```
## 6.1 Obtener connectionString
```
STORAGE_CONN=$(az storage account show-connection-string \
   --resource-group "$RG_NAME" \
   --name "$SA_NAME" \
   --query connectionString -o tsv)
```
# 7 Crear Blob Container
```
az storage container create \
   --name $BC_NAME \
   --connection-string "$STORAGE_CONN" \
   --public-access container
```
# 8 Subir fichero
```
az storage blob upload-batch \
   --destination $BC_NAME \
   --source /root/ \
   --pattern "config.json" \
   --connection-string "$STORAGE_CONN"
```
# 9 Ingresar VM
```
IP_PUBLICA=$(az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "publicIps" \
   -o tsv)
```

```
USER=$(az vm show \
   -d \
   --name $VM_NAME \
   --resource-group $RG_NAME \
   --query "osProfile.adminUsername" \
   -o tsv)
```

```
ssh $USER@$IP_PUBLICA
```
# 10 Crear contenedor
```
az login
PREFIX=datacenter
ACR_NAME="${PREFIX}acr10389"
REPO_NAME=$PREFIX/python-app
az acr login --name "$ACR_NAME"

docker run -d -p80:80 --name mi_proy  "${ACR_NAME}.azurecr.io/$REPO_NAME:latest"
```
# 11 Verificar
Abrimos un navegador y colocamos
```
http://IP_PUBLICA
```
# Anexo
### app.py
```
from flask import Flask
import json

app = Flask(__name__)

@app.route("/")
def home():
    with open("config.json", "r") as f:
        config = json.load(f)
    return f"Welcome to KKE Azure Labs: {config}"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```
### Dockerfile
```
FROM python:3.9-slim
WORKDIR /app
COPY app.py /app/
RUN pip install flask
CMD ["python", "app.py"]
```
