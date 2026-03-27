```
The Nautilus DevOps team is working on setting up secure SSH access for their virtual machines in Azure. One of the requirements is to add the SSH public key of the root user from the Azure client host (landing host) to the `datacenter-vm` Azure VM's `authorized_keys` file. This ensures secure and password-less SSH access to the VM.

### Task Details:

1) **VM Details**:

- The VM is named `datacenter-vm` and is running in the `eastus` region. The default SSH user is `azureuser` — use this user to connect to the VM.
- You need to add the root user's SSH public key from the Azure client host to the `authorized_keys` file of the VM's root user.
- The SSH public key of the root user on the Azure client host is located at `/root/.ssh/id_rsa.pub`.

2) **Public Key Addition**:

- Copy the public key located at `/root/.ssh/id_rsa.pub` on the Azure client host to the `authorized_keys` file of the root user on `datacenter-vm`.
- Ensure that the proper permissions for the `.ssh` folder and `authorized_keys` file are set on the VM.

3) **Verification**:

- After adding the public key, make sure that you are able to SSH into the `datacenter-vm` VM as the `root` user from the Azure client host without needing a password.

### Important Notes:

- Ensure that the VM is up and running before attempting to SSH.
- You may need to adjust the firewall or security group rules for the VM to allow SSH access.

  

Use the following Azure credentials to access the Azure portal:

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|
|Start Time|Fri Jan 30 13:44:47 UTC 2026|
|End Time|Fri Jan 30 14:44:47 UTC 2026|
```

Listar **Resource Groups**:
```
az group list
```

Capturar _name_ de **Resource Groups**:
```
RG_NAME=$(az group list --query [0].name --output tsv)
```

Listar **VM** que se encuentran en **Resource Groups**:
```
az vm list -g $RG_NAME
```

Capturar _id_ y _name_ de **VM**:
```
VM_ID=$(az vm list -g $RG_NAME --query [0].id --output tsv)
VM_NAME=$(az vm list -g $RG_NAME --query [0].name --output tsv)
```

Ver datalladamente **VM**:
```
az vm show -d --ids $VM_ID
```

Capturar _ip publica_ de **VM**:
```
IP_PUBLICA=$(az vm show -d --ids $VM_ID --query publicIps --output tsv)
```

Ingresar a **VM**:
```
ssh azureuser@$IP_PUBLICA 
```
Ingresa sin inconvenientes

Ingresar a **VM** con usuario _root_:
```
ssh root@$IP_PUBLICA
```
Y nos da este error:
```
Please login as the user "azureuser" rather than the user "root".

Connection to 172.212.187.200 closed.
```

Lo que debemos hacer es ingresar con _azureuser_ y luego cambiar a usuario _root_, y modificar el fichero _/root/.ssh/authorized_keys_ que por defecto tiene un comportamiento de seguridad estandar que bloquea explicitamente el acceso a _root_ y te obliga a entrar como _azureuser_.

```
ssh azureuser@$IP_PUBLICA
sudo -i
```

Editamos el fichero _/root/.ssh/authorized_keys_ 
```
vi /root/.ssh/authorized_keys
```
Borramos todo lo que haiga antes de **ssh-rsa**, por decir: no-port-forwarding,no-agent-forwarding...

```
no-port-forwarding,no-agent-forwarding,no-X11-forwarding,command="echo 'Please login as the user \"azureuser\" rather than the user \"root\".';echo;sleep 10;exit 142"
```

Ahora ingresamos con _root_:
```
ssh root@$IP_PUBLICA
```

NOTA: La configuracion del **host** ya esta configurada:
```
sudo vi /etc/ssh/sshd_config
```

```
PermitRootLogin yes
```
