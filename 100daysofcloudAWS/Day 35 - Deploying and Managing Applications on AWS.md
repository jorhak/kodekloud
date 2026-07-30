```
The Nautilus DevOps team needs a new private RDS instance for their application. They need to set up a MySQL database and ensure that their existing EC2 instance can connect to it. This will help in managing their database needs efficiently and securely.

1) Task Details:

Create a private RDS instance named datacenter-rds using a sandbox template.
The engine type must be MySQL v8.4.5, and it must be a db.t3.micro type instance.
The master username must be datacenter_admin with an appropriate password.
The RDS storage type must be gp2, and the storage size must be 5GiB.
Create a database named datacenter_db.
Keep the rest of the configurations as default. Ensure the instance is in available state.
Adjust the security groups so that the datacenter-ec2 instance can connect to the RDS on port 3306 and also open port 80 for the instance.
2) An EC2 instance named datacenter-ec2 exists. Connect to this instance from the AWS console. Create an SSH key (/root/.ssh/id_rsa) on the aws-client host if it doesn't already exist. Add the public key to the authorized keys of the root user on the EC2 instance for password-less SSH access.

3) There is a file named index.php under the /root directory on the aws-client host. Copy this file to the datacenter-ec2 instance under the /var/www/html/ directory. Make the appropriate changes in the file to connect to the RDS.

4) You should see a Connected successfully message in the browser once you access the instance using the public IP.


Use the below given AWS Credentials: (You can run the showcreds command on the aws-client host to retrieve these credentials)

Console URL	https://046467849567.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	
Start Time	Wed Jul 29 16:55:16 UTC 2026
End Time	Wed Jul 29 17:55:16 UTC 2026
Notes:

Create the resources only in the us-east-1 region.
To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
PREFIX=xfusion
RDS_NAME=$PREFIX-rds
ENGINE=mysql
ENGINE_VERSION=8.4.5
INSTANCE_CLASS=db.t3.micro
RDS_USERNAME=${PREFIX}_admin
RDS_PASSWORD=''
RDS_STORAGE_TYPE=gp2
ALLOCATED_STORAGE=5
DB_NAME=${PREFIX}_db
INSTANCE_NAME=$PREFIX-ec2
IMAGE=Ubuntu2404
REGION=us-east-1
SRC=/root/index.php
DEST=/var/www/html
```
# 1 Crear Instancia RDS
```
aws rds create-db-instance \
    --db-name $DB_NAME \
    --db-instance-identifier $RDS_NAME \
    --allocated-storage $ALLOCATED_STORAGE \
    --db-instance-class $INSTANCE_CLASS \
    --engine $ENGINE \
    --region $REGION \
    --engine-version $ENGINE_VERSION \
    --port 3306 \
    --master-username $RDS_USERNAME \
    --master-user-password "$RDS_PASSWORD" \
    --no-publicly-accessible \
    --no-multi-az \
    --storage-type $RDS_STORAGE_TYPE
```

```
aws rds wait db-instance-available \
    --db-instance-identifier $RDS_NAME
```
Abrimos una segunda terminal ejecutamos **Variables de entorno** y nos saltamos al paso 2
## 1.2 Verificar que se creo la instancia RDS
```
aws rds describe-db-instances \
    --db-instance-identifier $RDS_NAME \
    --query "DBInstances[].{Nombre:DBName,Estado:DBInstanceStatus,Identificador:DBInstanceIdentifier,Usuario:MasterUsername}" \
    --output table
```
## 1.3 Pasos para conexion
```
DB_ENDPOINT=$(aws rds describe-db-instances \
    --db-instance-identifier $RDS_NAME \
    --query "DBInstances[0].Endpoint.Address" \
    --output text)
echo $DB_ENDPOINT
```

Capturamos la URL que sera nuestro host para el paso 7.
## 1.4 Modificar RDS a publica (No se ejecuta)
```
aws rds modify-db-instance \
    --db-instance-identifier $RDS_NAME \
    --publicly-accessible \
    --apply-immediately
```

Volvemos al paso 7.
# 2 Configurar Security Group de la instancia
## 2.1 Obtener ID de Security Group
```
SG_ID=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].SecurityGroups[].GroupId" \
    --output text)
```
## 2.2 Habilitar puerto HTTP
```
aws ec2 authorize-security-group-ingress \
    --region $REGION \
    --group-id $SG_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```
# 3 Ingresar a servidor
## 3.1 Generar para de llaves
```
ssh-keygen -t rsa -b 4096
```
Copiamos la llave publica y la pegamos en authorized_keys luego de realizar el paso 3.2.
```
cat .ssh/id_rsa.pub
```
## 3.2 Ingresamos a la consola
EC2>Instances>$PREFIX-ec2 luego damos click sobre **Connect**, nos quedamos en la pestana **In web browser**, marcamos **EC2 Instance Connect**, en **Settings** la dejamos con IPv4, por ultimo damos click en **Connect**.
Agregamos la llave publica que copiamos en el paso 3.1:
```
sudo su
vi ~/.ssh/authorized_keys
exit
```

```
vi ~/.ssh/authorized_keys
```
Agregamos la llave publica para ambos usuarios.
### Nota:
Port 22 (SSH) is open to all IPv4 addresses

Port 22 (SSH) is currently open to all IPv4 addresses, indicated by 0.0.0.0/0 in the inbound rule in [your security group](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#SecurityGroup:groupId=sg-030e4e8e9cf20a4ad) . For increased security, consider restricting access to only the EC2 Instance Connect service IP addresses for your Region: 18.206.107.24/29.
## 3.3 Actualizar CIDR para SSH
No ejecutar estos comandos ya que no funcionaron, no me dejaron conectarme por ssh, saltar al paso 3.4
#### 3.3.1 Revocar regla
```
aws ec2 revoke-security-group-ingress \
    --region $REGION \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```
#### 3.3.2 Agregando nueva regla para SSH
```
aws ec2 authorize-security-group-ingress \
    --region $REGION \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 18.206.107.24/29
```

Volvemos a la consola de kodekloud
## 3.4 Capturar usuario
```
USER=ubuntu
##### o 
USER=root
```
## 3.5 Capturar IP publica
```
IP_PUBLIC=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].PublicIpAddress" \
    --output text)
```
## 3.6 Ingresar a Instancia
```
ssh $USER@$IP_PUBLIC
```

# Opcion A
# 4 Instalar Nginx
```
sudo apt update && sudo apt install nginx -y
```
# 4.1 Habilitar los permisos correspondientes
```
sudo chown -R www-data:www-data /var/www/html
sudo chmod o+w /var/www/html
exit
```
## 4.2 Subir fichero
```
scp $SRC $USER@$IP_PUBLIC:$DEST
```
# 5. Instalar php-fpm
Volvemos a ingresar al servidor
```
ssh $USER@$IP_PUBLIC
```
## 5.1 OPCIONAL
Podemos omitir estos comandos e ir directo al paso 5.2
```
sudo apt search php8.1 
sudo apt search php8.1-fpm
sudo apt info php8.1 
sudo apt info php8.1-fpm
```
## 5.2 Instalar php php-fpm php-cli
```
sudo apt install php8.1 php8.1-cli php8.1-fpm -y
```
## 5.3 Configurar php-fpm
### Verificar directorio
```
ls -la /var/run/php/
```
## 5.4 Editar pool principal php-fpm
```
sudo vi /etc/php/8.1/fpm/pool.d/www.conf
```

```
listen = /run/php/php8.1-fpm.sock
user = www-data
group = www-data	
listen.owner = www-data
listen.group = www-data
listen.mode = 0660	
```
#### Buscar directorio (por si acaso)
```
sudo find /etc -name "www.conf" 2>/dev/null
sudo find /var -type d -name php* 2>/dev/null
```
## 5.5 Habilitar y arrancar php-fpm
```
sudo systemctl status php8.1-fpm
sudo systemctl enable php8.1-fpm
sudo systemctl start php8.1-fpm
```

# 6 Configurar Nginx y PHP-FPM para que trabajen juntos
## 6.1 Antes sacamos un backup
```
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.backup
```
## 6.2 Configurar Nginx
```
sudo vi /etc/nginx/sites-available/default
```

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;


        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.php index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

        # pass PHP scripts to FastCGI server
        #
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;

                # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/run/php/php8.1-fpm.sock;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
                include fastcgi_params;
        }
        
        # Denegar el acceso a archivos ocultos (como .htaccess) 
        location ~ /\.ht { 
		        deny all; 
		}
}
```
## 6.2 Detener servicio Apache2
Hay un servicio utilizando el puerto 80 y en este caso es apache2, vamos a detenerlo porque nos causa conflicto con nginx.
```
sudo systemctl stop apache2
sudo systemctl disable apache2
```
## 6.3 Verificar sintaxis y reiniciar servicio
```
sudo nginx -t
sudo systemctl restart nginx
```

## 6.4 Reiniciar php-fpm
```
sudo systemctl restart php8.1-fpm
```

## 6.5 Asegurar permisos
```
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```
Nos vamos al paso 7.
# Opcion B
# 4 Verificar si Apache instalados
```
sudo systemctl status apache2
```
## 4.1 Habilitar permisos
```
sudo chown -R www-data:www-data /var/www/html
sudo chmod o+w /var/www/html
```
# 5 Instalar dependicias de PHP
```
sudo apt install php8.1 php8.1-mysql libapache2-mod-php8.1 -y
```
## 5.1 Cambiar el directorio index de Apache
```
sudo vi /etc/apache2/mods-enabled/dir.conf
```

```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

```
sudo systemctl restart apache2
sudo systemctl status apache2
```
# 5.2 Configurar Virtual Host
#### 5.2.1 Realizar backup
```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf.backup
```
#### 5.2.2 Configurar Virtual Host
```
sudo vi /etc/apache2/sites-available/000-default.conf
```

```
<VirtualHost *:80>
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
## 5.3 Salir de servidor
```
exit
```
# 6 Subir fichero
```
scp $SRC $USER@$IP_PUBLIC:$DEST
```
## 6.1 Ingresar al servidor
```
ssh $USER@$IP_PUBLIC
```
# 7 Modificar fichero index.php
Aqui volvemos al paso 1 en la terminal 1 que dejamos ejecutando y seguimos con el paso 1.2.
```
sudo vi /var/www/html/index.php
```

```
<?php
$dbname = 'xfusion_db';
$dbuser = 'xfusion_admin';
$dbpass = '';
$dbhost = 'xfusion-rds.cpyyqkuacqo5.us-east-1.rds.amazonaws.com';

$link = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");
mysqli_select_db($link, $dbname) or die("Could not open the db '$dbname'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($link, $test_query);

$tblCnt = 0;
while($tbl = mysqli_fetch_array($result)) {
  $tblCnt++;
}

if (!$tblCnt) {
  echo "Connected successfully<br />\n";
} else {
  echo "Connected successfully<br />\n";
}
?>
```
# 8 Verificacion
Ahora que tenemos todo configurado abrimos un navegador y colocamos la IP publica.
Desde la terminal, antes debemos ejecutar el comando del paso 3.5 en la terminal 1:
```
curl -i $IP_PUBLIC
```
## 8.1 Verificar que el servicio este funcionando
Ejecutamos estos comandos dentro del servidor terminal 2:
```
sudo tail -n 20 /var/log/nginx/access.log
sudo tail -n 20 /var/log/nginx/error.log
```
## 8.2 Cambiar de publica a privada RDS (No ejecutamos)
Terminal 1
```
aws rds modify-db-instance \
    --db-instance-identifier $RDS_NAME \
    --no-publicly-accessible \
    --apply-immediately
```
Aqui ya se --publicly-accessible o --no-publicly-accessible puedo realizar la consulta por curl o el navegador.

```
aws rds describe-db-instances \
    --db-instance-identifier $RDS_NAME \
    --query "DBInstances[*].[DBInstanceIdentifier,PubliclyAccessible,DBInstanceStatus]" \
    --output table
```

RDS instance 'devops-rds' is not using MySQL version 8.4.5.
RDS instance 'nautilus-rds' is not a private instance
SSH key is not configured for passwordless access to the instance
SSH key is not configured for passwordless access to the instance