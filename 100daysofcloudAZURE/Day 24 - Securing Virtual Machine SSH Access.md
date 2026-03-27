```
The Nautilus DevOps team needs to set up a new Virtual Machine (VM) on the Azure cloud that can be accessed securely from their landing host (`azure-client`). Follow the steps below to complete this task:

1. **Create an SSH Key**: On the `azure-client` host, check if an SSH key already exists. If it doesn’t exist, create a new SSH key on the `azure-client` host that will be used for password-less SSH access.
    
2. **Create a Virtual Machine**: Use the Azure Portal or Azure CLI to create a new Virtual Machine named `xfusion-vm` in the `westus` region. Set the VM size to **Standard_B1s** and configure the VM with **SSH access** for the `azureuser` account using the newly created SSH key.
    
3. **Configure SSH Access**: Ensure that the SSH key from the `azure-client` host is added to the `azureuser` account on `xfusion-vm`, enabling secure, password-less SSH access from the `azure-client` host.
    
4. **Verify Connectivity**: Test the connection from `azure-client` to `xfusion-vm` using SSH to confirm that password-less access has been set up correctly.
    

Complete these tasks entirely within the Azure Portal or Azure CLI.

  

Use the below given **Azure Credentials:** (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Mon Mar 02 18:15:42 UTC 2026|
|End Time|Mon Mar 02 19:15:42 UTC 2026|

`Notes:`

- Create the resources only in `westus` region.
    
- To `display` or `hide` the terminal of the Azure client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Variables de entorno:
```
VM_NAME=devops-vm
VM_SIZE=Standard_B1s
LOCATION=westus
IMAGE="Canonical:ubuntu-24_04-lts:server:latest"
```

1. Crear llave **SSH**:
```
ssh-keygen -t rsa -b 4096
```

2. Crear **VM**:
- Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

- Crear **VM**:
```
az vm create \
    --resource-group $RG_NAME \
    --name $VM_NAME \
    --image $IMAGE \
    --size Standard_B1s \
    --storage-sku Standard_LRS \
    --location $LOCATION \
    --ssh-key-values /root/.ssh/id_rsa.pub
```

- Capturar **IP Publica**:
```
IP_ADDRESS=$(az vm show --show-details --resource-group $RG_NAME --name $VM_NAME --query publicIps --output tsv)
```

3. Verificar **NSG**:
```
az network nsg list
```

4. Ver reglas
```
az network nsg rule list \
   -g $RG_NAME \
   --nsg-name devops-vmNSG
```

5. Verificar conexion por **SSH**:
```
ssh -i .ssh/id_rsa azureuser@$IP_ADDRESS
```

6. Estado de **VM**:
```
az vm show -g $RG_NAME -n $VM_NAME -d --query "{NOMBRE:name,IP_PRIVADA:privateIps,IP_PUBLICA:publicIps,ESTADO:powerState}" --output table
```
