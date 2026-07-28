```
The Nautilus Development Team is preparing for a major update to their database infrastructure. To ensure a smooth transition and to safeguard data, the team has requested the DevOps team to take a snapshot of the current RDS instance and restore it to a new instance. This process is crucial for testing and validation purposes before the update is rolled out to the production environment. The snapshot will serve as a backup, and the new instance will be used to verify that the backup process works correctly and that the application can function seamlessly with the restored data.

As a member of the Nautilus DevOps Team, your task is to perform the following:

Take a Snapshot: Take a snapshot of the devops-rds RDS instance and name it devops-snapshot (please wait devops-rds instance to be in available state).

Restore the Snapshot: Restore the snapshot to a new RDS instance named devops-snapshot-restore.

Configure the New RDS Instance: Ensure that the new RDS instance has a class of db.t3.micro.

Verify the New RDS Instance: The new RDS instance must be in the Available state upon completion of the restoration process.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL https://541882316995.signin.aws.amazon.com/console?region=us-east-1
Username kk_labs_user
Password contra
Start Time Fri Jul 17 02:07:26 UTC 2026
End Time Fri Jul 17 03:07:26 UTC 2026

Notes:
Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:

toggle button
```
# Variables de entorno
```
PREFIX=xfusion
RDS_NAME=$PREFIX-rds
SNAPSHOT_NAME=$PREFIX-snapshot
RDS_NEW_NAME=$PREFIX-snapshot-restore
INSTANCE_CLASS=db.t3.micro
PORT=3306
REGION=us-east-1
ENGINE_VERSION=8.4.5
USERNAME="$PREFIX_admin"
SNAPSHOT_TYPE=manual
```
# Crear Snapshot
#### Ver si la instancia RDS esta habilitada
Hay que esperar entre 3 y 5 minutos para que se cree la instancia RDS a la que se le va realizar un Snapshot
```
aws rds describe-db-instances \
    --db-instance-identifier $RDS_NAME \
    --query "DBInstances[0].DBInstanceStatus" \
    --output text
```
#### Creamos Snapshot
```
aws rds create-db-snapshot \
    --db-snapshot-identifier $SNAPSHOT_NAME \
    --db-instance-identifier $RDS_NAME \
    --tags Key=ENV,Value=DEV \
    --region $REGION
```
#### Ver si se creo Snapshot
```
aws rds describe-db-snapshots \
    --db-snapshot-identifier $SNAPSHOT_NAME \
    --db-instance-identifier $RDS_NAME \
    --snapshot-type $SNAPSHOT_TYPE \
    --query "DBSnapshots[].{IdentiSnapshot:DBSnapshotIdentifier,IdentiInstance:DBInstanceIdentifier,Estado:Status}" \
    --output table
```
# 2 Restaurar Snapshot
```
aws rds restore-db-instance-from-db-snapshot \
    --db-instance-identifier $RDS_NEW_NAME \
    --db-snapshot-identifier $SNAPSHOT_NAME \
    --db-instance-class $INSTANCE_CLASS \
    --port $PORT \
    --no-multi-az \
    --publicly-accessible \
    --tags Key=ENV,Value=DEV \
    --storage-type gp2 \
    --region $REGION
```
Aqui vamos a esperar entre 3 y 5 minutos
```
aws rds wait db-instance-available \
    --db-instance-identifier $RDS_NEW_NAME
```
#### Modificar password
```
aws rds modify-db-instance \
    --db-instance-identifier $RDS_NEW_NAME \
    --master-user-password '$$Joaquina14$$'
```
# 3 Verificar que se creo la nueva RDS desde Snapshot
```
aws rds describe-db-instances \
    --db-instance-identifier $RDS_NEW_NAME \
    --query "DBInstances[].{Nombre:DBName,Estado:DBInstanceStatus,Identificador:DBInstanceIdentifier,Usuario:MasterUsername}" \
    --output table
```
#### Pasos de conexion
```
DB_ENDPOINT=$(aws rds describe-db-instances \
    --db-instance-identifier $RDS_NEW_NAME \
    --query "DBInstances[0].Endpoint.Address" \
    --output text)
echo $DB_ENDPOINT
```
# Nos vamos a KillerCoda y creamos un container para conectarnos a la instancia RDS
Crear container y ingresar
```
dig +short <DB_ENDPOINT>
host <DB_ENDPOINT>
docker run -d -p3306:3306 --name misql -e MYSQL_ROOT_PASSWORD=toor mysql
docker exec -it misql bash
```
Ejecutamos los comandos dentro del container
```
curl -o global-bundle.pem 
chmod 644 ./global-bundle.pem
curl ifconfig.me
```

Volvemos a KodeKloud
Capturar ID Security Group
```
SG_RDS=$(aws rds describe-db-instances \
    --db-instance-identifier $RDS_NEW_NAME \
    --query "DBInstances[0].VpcSecurityGroups[0].VpcSecurityGroupId" \
    --output text)
```
Agregar la IP de KillerCoda
```
aws ec2 authorize-security-group-ingress \
    --group-id "$SG_RDS" \
    --protocol tcp \
    --port 3306 \
    --cidr "212.2.242.179/32"
```

Volvemos a KillerCoda
Nos conectamos a la instancia RDS
```
PREFIX=xfusion
USERNAME="${PREFIX}_admin"
HOST=xfusion-snapshot-restore.c69vagzcl0in.us-east-1.rds.amazonaws.com

mysql -h $HOST -P 3306 -u $USERNAME -p --ssl-mode=VERIFY_IDENTITY --ssl-ca=./global-bundle.pem
```