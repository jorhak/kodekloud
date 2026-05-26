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
VM_MYSQL_NAME=datacenter-mysql-vm
VM_MYSQL_SIZE=Standard_B1s
VM_MYSQL_STORAGE_SKU=Standard_LRS
MYSQL_REGION=centralus
MYSQL_USERNAME=datacenter_admin
MYSQL_PASSWORD=Namin@123456
MYSQL_PORT='3306'
VM_PHP_NAME=datacenter-php-vm
REGION=eastus
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
### Opcion 1
Para crear esta **VM** debemos contar con los permisos necesarios. Nos vamos a ir por la **Opcion 2**.
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
### Opcion 2
Ya que no contamos con los permisos necesarios vamos a utilizar otra imagen.
##### Script de configuracion de DB
```
vi install.sh
```

```
#!/bin/bash
sudo apt-get update
sudo apt-get install -y mysql-server
sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mysql/mysql.conf.d/mysqld.cnf
sudo systemctl restart mysql
sudo mkdir -p /jet
echo -e '#!/bin/bash\nsudo mysql' | sudo tee /jet/enter
sudo chmod +x /jet/enter
echo "CREATE DATABASE IF NOT EXISTS datacenter_db;
CREATE USER IF NOT EXISTS 'datacenter_user'@'%' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON datacenter_db.* TO 'datacenter_user'@'%';
FLUSH PRIVILEGES;" | sudo /jet/enter
```
##### Buscar imagen
```
az vm image list \
  --publisher "Canonical" \
  --offer "0001-com-ubuntu-server-jammy" \
  --sku "22_04-lts-gen2" \
  --output table
```
#### Crear VM
```
az vm create \
  --resource-group $RG_NAME \
  --name $VM_MYSQL_NAME \
  --image "Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest" \
  --admin-username $MYSQL_USERNAME \
  --admin-password $MYSQL_PASSWORD \
  --authentication-type password \
  --size $VM_MYSQL_SIZE \
  --storage-sku $VM_MYSQL_STORAGE_SKU \
  --custom-data install.sh \
  --location $MYSQL_REGION
```

```
az vm wait \
   --name $VM_MYSQL_NAME \
   --resource-group $RG_NAME \
   --created
```
### Abrir puerto
```
az vm open-port \
  --resource-group $RG_NAME \
  --name $VM_MYSQL_NAME \
  --port 3306 \
  --priority 1001
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

```
echo -e "\e[33mUsuario::$USER\e[0m \n\e[34mIP Publica::$IP_PUBLIC\e[0m"
```
# 3 Crear Base de Datos
```
ssh $USER@$IP_PUBLIC
```
#### Opcion 1
Implementamos solo si tenemos en el **Paso 1>Opcion 1**.
```
sudo /jet/enter mysql
```

```
CREATE DATABASE datacenter_db;
CREATE USER 'datacenter_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON datacenter_db TO 'datacenter_user'@'localhost';
FLUSH PRIVILEGES;
```
# 4 Ingresar a VM PHP
Abrimos otra terminal y ejecutamos **Variables de entorno** y **Obtener Resource Group**.
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
echo -e "\e[32mUsuario::$USER_PHP\e[0m \n\e[31mIP Publica::$IP_PUBLIC_PHP\e[0m"
```

```
ssh $USER_PHP@$IP_PUBLIC_PHP
```
# Configurar fichero /var/www/html/db_test.php
```
sudo vi /var/www/html/db_test.php
```
### Before
```
<?php
    $servername = "<mysql-vm-public-ip>";
    $username = "nautilus_user";
    $password = "password123";
    $dbname = "nautilus_db";

    // Create connection
    $conn = new mysqli($servername, $username, $password, $dbname);

    // Check connection
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }
    echo "Connected successfully";
?>
```
### After
```
<?php
    $servername = "130.131.243.197";
    $username = "datacenter_user";
    $password = "password123";
    $dbname = "datacenter_db";

    // Create connection
    $conn = new mysqli($servername, $username, $password, $dbname);

    // Check connection
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }
    echo "Connected successfully";
?>
```
# Configurar 
```
sudo vi /etc/apache2/mods-enabled/dir.conf
```
#### Before
```
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```
#### After
```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
#### Reiniciar servicio
```
sudo systemctl restart apache2
```
# Cambiar nombre del fichero
```
sudo mv /var/www/html/index.php /var/www/html/db_test.php
```
# Verificar
```
curl -i $IP_PUBLIC_PHP/db_test.php
```

```
Failed to access the PHP test page at 'http://20.172.227.34/db_test.php'.
```