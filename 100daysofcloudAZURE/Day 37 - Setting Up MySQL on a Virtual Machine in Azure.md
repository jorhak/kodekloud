```
The Nautilus DevOps team is tasked with integrating a PHP application hosted on an Azure VM with a MySQL database hosted on another Azure VM. This will validate the application's ability to connect to the database in the cloud.

1. **Create the MySQL VM**:
    
    - Create a VM named `xfusion-mysql-vm` using the **MySQL Jetware** image from the Azure Marketplace.
    - Configure the VM in the **Central US** region.
    - Use `Password` as the authentication type.
    - Set the **username** as `xfusion_admin` and the **password** as `xxx`.
    - Allow inbound traffic on port `3306` to enable MySQL access.
2. **Setup the MySQL Database**:
    
    - SSH into the `xfusion-mysql-vm`.
    - Use the `sudo /jet/enter mysql` command to access the MySQL shell.
    - Create a database named `xfusion_db`.
    - Create a MySQL user named `xfusion_user` with password `xxx`.
    - Grant all privileges on the `xfusion_db` database to this user.
3. **PHP VM Setup**:
    
    - A VM named `xfusion-php-vm` already exists in the **East US** region.
    - This VM is hosting a PHP application and contains a pre-existing `db_test.php` file in the `/var/www/html/` directory.
4. **Database Connection Configuration**:
    
    - Retrieve the **public IP address** of the `xfusion-mysql-vm`.
    - Update the database connection settings in the `db_test.php` file to use the MySQL credentials and public IP address of the `xfusion-mysql-vm`.
5. **Validation**:
    
    - Access the `db_test.php` file from the `xfusion-php-vm` using its public IP address.
    - Ensure the file displays the message `Connected successfully`, confirming the connection between the PHP application and the MySQL database.

  

Use the Azure Portal URL and login credentials below:

|Portal URL|[https://portal.azure.com](https://portal.azure.com/)|
|---|---|
|Username|[kk_lab_user_main@azureprod.onmicrosoft.com]|
|Password|contra|

  
`Notes:`

- Ensure the MySQL database allows inbound traffic on port `3306`.
- Verify that the PHP application on the `xfusion-php-vm` successfully connects to the MySQL database on the `xfusion-mysql-vm`.
```
# Variables de entorno
```
VM_MYSQL_NAME=xfusion-mysql-vm
VM_MYSQL_SIZE=Standard_B1s
VM_MYSQL_STORAGE_SKU=Standard_LRS
MYSQL_REGION=centralus
MYSQL_USERNAME=xfusion_admin
MYSQL_PASSWORD=
MYSQL_PORT='3306'
VM_PHP_NAME=xfusion-php-vm
REGION=eastus
```

```
Security type: Standard
Image:Percona Server for MySQL 5.7 on Ubuntu -64 Gen1
VM architecture: x64
Size: Standard_D4s_v3 - 4 vcpus, 16 GiB memory ($160)
Size: Standard_B2s - 2 vcpus, 4 GiB memory ($36.43)
Username: azureuser
SSH public key source: Generate new key pair
SSH Key type: RSA SSH format
```

# Obtener Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# 1 Buscar imagen sugerida
```
az vm image list \
  --all \
  --publisher "jetware-srl" \
  --offer "percona_mysql" \
  --sku "percona_mysql57-ubuntu-1604" \
  --output table
```

```
az vm image terms accept \
  --publisher "jetware-srl" \
  --offer "percona_mysql" \
  --plan "percona_mysql57-ubuntu-1604" \
  --urn "jetware-srl:percona_mysql:percona_mysql57-ubuntu-1604:1.0.170503"
```

```
az vm image terms accept \
  --urn "jetware-srl:percona_mysql:percona_mysql57-ubuntu-1604:1.0.170503"
```
# 2 Crear VM
```
az vm create \
  --resource-group $RG_NAME \
  --name $VM_MYSQL_NAME \
  --image "jetware-srl:percona_mysql:percona_mysql57-ubuntu-1604:1.0.170503" \
  --admin-username $MYSQL_USERNAME \
  --admin-password $MYSQL_PASSWORD \
  --authentication-type password \
  --size $VM_MYSQL_SIZE \
  --storage-sku $VM_MYSQL_STORAGE_SKU \
  --public-ip-sku Standard \
  --location $MYSQL_REGION
```

```
USER=$(az vm show \
   -n $VM_MYSQL_NAME \
   -g $RG_NAME \
   -d \
   --query osProfile.adminUsername \
   --output tsv)
```

```
IP_PUBLIC=$(az vm show \
   -n $VM_MYSQL_NAME \
   -g $RG_NAME \
   -d \
   --query publicIps \
   --output tsv)
```
# 3 Crear Base de Datos
```
ssh $USER@$IP_PUBLIC
```

```
sudo /jet/enter mysql
```

```
CREATE DATABASE xfusion_db;
CREATE USER 'xfusion_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON xfusion_db TO 'xfusion_user'@'localhost';
FLUSH PRIVILEGES;
```
# 4 Ingresar a VM PHP
```
USER_PHP=$(az vm show \
   -n $VM_PHP_NAME \
   -g $RG_NAME \
   -d \
   --query osProfile.adminUsername \
   --output tsv)
```

```
IP_PUBLIC_PHP=$(az vm show \
   -n $VM_PHP_NAME \
   -g $RG_NAME \
   -d \
   --query publicIps \
   --output tsv)
```

```
ssh $USER_PHP@$IP_PUBLIC_PHP
```
# Configurar fichero /var/www/html/db_test.php
```
vi /var/www/html/db_test.ph
```

```

```
# Verificar
```
curl -i $IP_PUBLIC_PHP
```
