```
The Nautilus Development Team is working on a new application feature that requires a reliable and scalable database solution. To facilitate development and testing, they need a new private RDS instance. This instance will be used to store critical application data and must be provisioned using the AWS free tier to minimize costs during the initial development phase. The team has chosen MySQL as the database engine due to its compatibility with their existing systems. The DevOps team has been tasked with setting up this RDS instance, ensuring that it is correctly configured and available for use by the development team.

As a member of the Nautilus DevOps Team, your task is to perform the following:

Provision a Private RDS Instance: Create a new private RDS instance named devops-rds using the Full configuration database creation method, and select the Free tier template. Further, it must be a db.t3.micro type instance.

Engine Configuration: Use the MySQL engine with version 8.4.x.

Enable Storage Autoscaling: Enable storage autoscaling and set the threshold value to 50GB. Keep the rest of the configurations as default.

Instance Availability: Ensure the instance is in the available state before submitting this task.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL https://374577096375.signin.aws.amazon.com/console?region=us-east-1
Username kk_labs_user
Password contra
Start Time Thu Jul 16 18:40:18 UTC 2026
End Time Thu Jul 16 19:40:18 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:

toggle button
```
# Variables de entorno
```
RDS_NAME=datacenter-rds
INSTANCE_CLASS=db.t3.micro
SGDB=mysql
ENGINE_VERSION=8.4.10
MAX_ALLOCATED_STORAGE=50
REGION=us-east-1
```
# 1 Crear instancia RDS
```
aws rds create-db-instance \
    --db-name mydbtest \
    --db-instance-identifier $RDS_NAME \
    --allocated-storage 20 \
    --max-allocated-storage $MAX_ALLOCATED_STORAGE \
    --db-instance-class $INSTANCE_CLASS \
    --engine $SGDB \
    --region $REGION \
    --engine-version $ENGINE_VERSION \
    --port 3306 \
    --master-username jorhak \
    --master-user-password '$$Joaquina14$$' \
    --no-publicly-accessible \
    --no-multi-az \
    --storage-type gp3
```

```
aws rds wait db-instance-available \
    --db-instance-identifier $RDS_NAME
```
# 2 Verificar que se creo la instancia RDS
```
aws rds describe-db-instances \
    --db-instance-identifier $RDS_NAME \
    --query "DBInstances[].{Nombre:DBName,Estado:DBInstanceStatus,Identificador:DBInstanceIdentifier,Usuario:MasterUsername}" \
    --output table
```
#### Pasos para la conexion
```
DB_ENDPOINT=$(aws rds describe-db-instances \
    --db-instance-identifier $RDS_NAME \
    --query "DBInstances[0].Endpoint.Address" \
    --output text)
echo $DB_ENDPOINT
```

```
dig +short $DB_ENDPOINT
host $DB_ENDPOINT
```
## Nos vamos a KillerCoda y creamos un container para conectarnos a la instancia RDS
Vemos si tenemos llegada y tambien vemos la IP
```
dig +short <DB_ENDPOINT>
host <DB_ENDPOINT>
```

Crear container y ingresar
```
docker run -d -p3306:3306 --name misql -e MYSQL_ROOT_PASSWORD=toor mysql
docker exec -it misql bash
```

Ejecutamos estos comandos dentro del container
```
curl -o global-bundle.pem 
chmod 644 ./global-bundle.pem
curl ifconfig.me
```

Volvemos a KodeKloud y agregamos la IP para poder tener conexion.
Primero obtenemos el ID Security Group
```
SG_RDS=$(aws rds describe-db-instances \
    --db-instance-identifier $RDS_NAME \
    --query "DBInstances[0].VpcSecurityGroups[0].VpcSecurityGroupId" \
    --output text)
```
Segundo agregamos la IP de KillerCoda para que se puedan conectar
```
aws ec2 authorize-security-group-ingress \
    --group-id "$SG_RDS" \
    --protocol tcp \
    --port 3306 \
    --cidr "212.2.242.179/32"
```
Tercero hacemos publica la instancia RDS
```
aws rds modify-db-instance \
    --db-instance-identifier $RDS_NAME \
    --publicly-accessible \
    --apply-immediately
```

Volvemos a KillerCoda y nos conectamos a la instancia RDS
```
mysql -h datacenter-rds.crrqx0icyfu9.us-east-1.rds.amazonaws.com -P 3306 -u jorhak -p --ssl-mode=VERIFY_IDENTITY --ssl-ca=./global-bundle.pem
```
# Opcional
Si tenemos un equipo dentro de la misma subnet privada colocamos la IP Privada del servidor
```
aws ec2 authorize-security-group-ingress \
    --group-id "$SG_RDS" \
    --protocol tcp \
    --port 3306 \
    --cidr "<IP_PRIVATE>/32"
```