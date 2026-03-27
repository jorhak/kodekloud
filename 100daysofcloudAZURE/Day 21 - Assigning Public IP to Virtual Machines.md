```
The Nautilus DevOps Team has received a new request from the Development Team to set up a new Azure Virtual Machine (VM). This VM will be used to host a new application that requires a stable public IP address. To ensure that the VM has a consistent public IP, a Static Public IP address needs to be associated with it. The VM will be named `devops-vm`, and the Static Public IP will be named `devops-pip`. This setup will help the Development Team to have a reliable and consistent access point for their application.

1. Create an Azure VM named `devops-vm` using any available Ubuntu image, with the VM size `Standard_B1s`.
2. Generate an SSH public key on the `azure-client` host and associate it with the VM for SSH access.
3. Associate a Static Public IP address named `devops-pip` with this VM.
4. Ensure the VM is accessible via SSH using the generated public key.

  

Use below given **Azure Credentials:** (You can run the `showcreds` command on the `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Thu Feb 26 13:02:28 UTC 2026|
|End Time|Thu Feb 26 14:02:28 UTC 2026|

  
`Notes:`

- Perform all operations in the `Central US` region.
```

Variables de entorno:
```
VM_NAME=xfusion-vm
VM_SIZE=Standard_B1s
SP_IP_NAME=xfusion-pip
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
USERNAME=azureuser
LOCATION=centralus
```

1. Listar **Resource Groups**:
```
az group list
```

2. Obtener _name_:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

3. Crear IP Publica:
```
az network public-ip create \
    --resource-group $RG_NAME \
    --name $SP_IP_NAME \
    --sku Standard \
    --allocation-method Static \
    --location $LOCATION
```

4. Generar llave SSH
```
ssh-keygen -t rsa -b 4096
```

5. Crear **VM**:
```
az vm create \
    --resource-group $RG_NAME \
    --name $VM_NAME \
    --image "$IMAGE" \
    --size $VM_SIZE \
    --public-ip-address $SP_IP_NAME \
    --location $LOCATION \
    --storage-sku Standard_LRS \
    --ssh-key-values /root/.ssh/id_rsa.pub
```

```
az vm open-port \
    --resource-group "$RG_NAME" \
    --name "$VM_NAME" \
    --port 22 \
    --priority 1001
```

5. Verificar **VM**:
```
az vm show -g $RG_NAME -n $VM_NAME -d --query "{NOMBRE:name,IP_PRIVADA:privateIps,IP_PUBLICA:publicIps,ESTADO:powerState}" --output table
```

6. Verificar **Static IP**:
```
az network public-ip show -n $SP_IP_NAME -g $RG_NAME --query "{NAME:name,IP_PUBLICA:ipAddress}" --output table
```

7. Obtener **Static IP**:
```
IP_PUBLICA=$(az network public-ip show -n $SP_IP_NAME -g $RG_NAME --query ipAddress --output tsv)
```

8. Conectar **SSH**:
```
ssh -i ./ssh/id_rsa $USERNAME@$IP_PUBLICA
```
