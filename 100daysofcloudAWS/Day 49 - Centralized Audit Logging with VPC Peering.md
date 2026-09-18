```
The Nautilus DevOps team needs to build a secure and scalable log aggregation setup within their AWS environment. The goal is to gather log files from an internal EC2 instance running in a private VPC, transfer them securely to another EC2 instance in a public VPC, and then push those logs to a secure S3 bucket.

1) A VPC named datacenter-priv-vpc already exists with a private subnet named datacenter-priv-subnet, a route table named datacenter-priv-rt, and an EC2 instance named datacenter-priv-ec2 (using ubuntu image). This instance uses the SSH key pair datacenter-key.pem already available on the AWS client host at /root/.ssh/.

2) Your task is to:

Create a new VPC named datacenter-pub-vpc.
Create a subnet named datacenter-pub-subnet and a route table named datacenter-pub-rt under this public VPC.
Attach an internet gateway to datacenter-pub-vpc and configure the public route table to enable internet access.
Launch an EC2 instance named datacenter-pub-ec2 into the public subnet using the same key pair as the private instance.
Create an IAM role named datacenter-s3-role with PutObject permission to an S3 bucket and attach it to the public EC2 instance.
Create a new private S3 bucket named datacenter-s3-logs-370783295.
Configure a VPC Peering named datacenter-vpc-peering between the private and public VPCs.
Modify both datacenter-priv-rt and datacenter-pub-rt to route each other's CIDR blocks through the peering connection.
On the private instance, configure a cron job to push the /var/log/boots.log file to the public instance (using scp or rsync).
On the public instance, configure a cron job to push that same file to the created S3 bucket.
The uploaded file must be stored in the S3 bucket under the path datacenter-priv-vpc/boot/boots.log.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://198341928488.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Mon Sep 14 17:20:25 UTC 2026
End Time	Mon Sep 14 18:20:25 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```bash
PREFIX=datacenter
VPC_PRIVATE_NAME=$PREFIX-priv-vpc
SUBNET_PRIVATE_NAME=$PREFIX-priv-subnet
RT_PRIVATE_NAME=$PREFIX-priv-rt
INSTANCE_PRIVATE_NAME=$PREFIX-priv-ec2
KEY_PAIR_NAME=$PREFIX-key.pem
KEY_PAIR_PATH=/root/.ssh/$KEY_PAIR_NAME
VPC_PUBLIC_NAME=$PREFIX-pub-vpc
SUBNET_PUBLIC_NAME=$PREFIX-pub-subnet
RT_PUBLIC_NAME=$PREFIX-pub-rt
IGW_PUBLIC_NAME=$PREFIX-pub-igw
INSTANCE_PUBLIC_NAME=$PREFIX-pub-ec2
ROLE_NAME=$PREFIX-s3-role
S3_NAME=$PREFIX-s3-logs-927609959
PEERING_NAME=$PREFIX-vpc-peering
S3_PATH=$PREFIX-priv-vpc/boot/boots.log
REGION=us-east-1
ID_IMAGE=ami-0b6d9d3d33ba97d99
INSTANCE_TYPE=t2.micro
SG_NAME=$PREFIX-sg
VPC_CIDR_PUBLIC="10.0.0.0/16"
SUBNET_CIDR_PUBLIC="10.0.1.0/24"
```
# Obtener los ID's de los recursos existentes
## ID VPC Private
```bash
ID_VPC_PRIVATE=$(aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=$VPC_PRIVATE_NAME \
    --query "Vpcs[0].VpcId" \
    --output text)
echo -e "\033[0;32mID VPC PRIVATE\033[0m \033[1;31m$ID_VPC_PRIVATE\033[0m"
```
# CIDR VPC Private
```bash
VPC_CIDR_PRIVATE=$(aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=$VPC_PRIVATE_NAME \
    --query "Vpcs[0].CidrBlock" \
    --output text)
echo -e "\033[0;32mVPC CIDR PRIVATE\033[0m \033[1;31m$VPC_CIDR_PRIVATE\033[0m"
```
## ID SUBNET Private
```bash
ID_SUBNET_PRIVATE=$(aws ec2 describe-subnets \
    --filters Name=tag:Name,Values=$SUBNET_PRIVATE_NAME \
    --query "Subnets[0].SubnetId" \
    --output text)
echo -e "\033[0;32mID SUBNET PRIVATE\033[0m \033[1;31m$ID_SUBNET_PRIVATE\033[0m"
```
## CIDR SUBNET Private
```bash
SUBNET_CIDR_PRIVATE=$(aws ec2 describe-subnets \
    --filters Name=tag:Name,Values=$SUBNET_PRIVATE_NAME \
    --query "Subnets[0].CidrBlock" \
    --output text)
echo -e "\033[0;32mSUBNET CIDR PRIVATE\033[0m \033[1;31m$SUBNET_CIDR_PRIVATE\033[0m"
```
## ID ROUTE TABLE Private
```bash
ID_RT_PRIVATE=$(aws ec2 describe-route-tables \
    --filters Name=tag:Name,Values=$RT_PRIVATE_NAME \
    --query "RouteTables[0].RouteTableId" \
    --output text)
echo -e "\033[0;32mID ROUTE TABLE PRIVATE\033[0m \033[1;31m$ID_RT_PRIVATE\033[0m"
```
## ID INSTANCE Private
```bash
ID_INSTANCE_PRIVATE=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_PRIVATE_NAME \
    --query "Reservations[0].Instances[0].InstanceId" \
    --output text)
echo -e "\033[0;32mID INSTANCE PRIVATE\033[0m \033[1;31m$ID_INSTANCE_PRIVATE\033[0m"
```
# 1 Crear VPC Public
```bash
VPC_ID=$(aws ec2 create-vpc \
    --cidr-block "$VPC_CIDR_PUBLIC" \
    --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value='$VPC_PUBLIC_NAME'}]" \
    --region $REGION \
    --query "Vpc.VpcId" \
    --output text)
echo -e "\033[0;32mID VPC PUBLIC\033[0m \033[1;31m$VPC_ID\033[0m"
```
# OPCIONAL solo si se pierde la conexion
```bash
VPC_ID=$(aws ec2 describe-vpcs \
    --filters "Name=tag:Name,Values=$VPC_PUBLIC_NAME" \
    --query "Vpcs[0].VpcId" \
    --output text)
echo -e "\033[0;32mID VPC PUBLIC\033[0m \033[1;31m$VPC_ID\033[0m"
```
# 2 Crear SUBNET Public
```bash
SUBNET_ID=$(aws ec2 create-subnet \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value='$SUBNET_PUBLIC_NAME'}]" \
    --cidr-block "$SUBNET_CIDR_PUBLIC" \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "Subnet.SubnetId" \
    --output text)
echo -e "\033[0;32mID SUBNET PUBLIC\033[0m \033[1;31m$SUBNET_ID\033[0m"
```
# OPCIONAL solo si se pierde la conexion
```bash
SUBNET_ID=$(aws ec2 describe-subnets \
    --filters "Name=tag:Name,Values=$SUBNET_PUBLIC_NAME" \
    --query "Subnets[0].SubnetId" \
    --output text)
echo -e "\033[0;32mID SUBNET PUBLIC\033[0m \033[1;31m$SUBNET_ID\033[0m"
```
## 2.1 Habilitar IP Public
```bash
aws ec2 modify-subnet-attribute \
    --map-public-ip-on-launch \
    --subnet-id $SUBNET_ID \
    --region $REGION
```
# 3 Crear IGW
## 3.1 Crear INTERNET GATEWAY
```bash
IGW_ID=$(aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value='$IGW_PUBLIC_NAME'}]" \
    --region $REGION \
    --query "InternetGateway.InternetGatewayId" \
    --output text)
echo -e "\033[0;32mID IGW PUBLIC\033[0m \033[1;31m$IGW_ID\033[0m"
```
# OPCIONAL solo si se pierde la conexion
```bash
IGW_ID=$(aws ec2 describe-internet-gateways \
    --filters "Name=tag:Name,Values=$IGW_PUBLIC_NAME" \
    --query "InternetGateways[0].InternetGatewayId" \
    --output text)
echo -e "\033[0;32mID IGW PUBLIC\033[0m \033[1;31m$IGW_ID\033[0m"
```
#### 3.1.1 Asocicar IWG con VPC Public
```bash
aws ec2 attach-internet-gateway \
    --internet-gateway-id $IGW_ID \
    --vpc-id $VPC_ID \
    --region $REGION
```
# 4 Crear ROUTE TABLE
```bash
RT_ID=$(aws ec2 create-route-table \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value='$RT_PUBLIC_NAME'}]" \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "RouteTable.RouteTableId" \
    --output text)
echo -e "\033[0;32mID RT PUBLIC\033[0m \033[1;31m$RT_ID\033[0m"
```
# OPCIONAL solo si se pierde la conexion
```bash
RT_ID=$(aws ec2 describe-route-tables \
    --filters "Name=tag:Name,Values=$RT_PUBLIC_NAME" \
    --query "RouteTables[0].RouteTableId" \
    --output text)
echo -e "\033[0;32mID RT PUBLIC\033[0m \033[1;31m$RT_ID\033[0m"
```
## 4.1 Crear Ruta
```bash
aws ec2 create-route \
    --route-table-id $RT_ID \
    --destination-cidr-bloc 0.0.0.0/0 \
    --gateway-id $IGW_ID \
    --region $REGION
```
## 4.2 Adjuntar Subnet publica a Route Table
```bash
aws ec2 associate-route-table \
    --subnet-id $SUBNET_ID \
    --route-table-id $RT_ID \
    --region $REGION
```
# 5 Crear INSTANCIA Public
## 5.1 Crear KEY PAIR
### 5.1.1 Capturamos la llave publica a partir del fichero $KEY_PAIR_PATH
```bash
sudo chmod 400 $KEY_PAIR_PATH
```

```bash
ssh-keygen -y -f $KEY_PAIR_PATH -cC "root@aws-client" > clave_publica.pub
ssh-keygen -y -f $KEY_PAIR_PATH -C "root@aws-client" > clave_publica.pub
```
### 5.1.2 Importar llave publica
```bash
aws ec2 import-key-pair \
    --key-name $KEY_PAIR_NAME \
    --public-key-material fileb://clave_publica.pub
```
## 5.2 Crear instancia
### 5.2.1 Crear Security Group
```bash
SG_ID=$(aws ec2 create-security-group \
    --group-name $SG_NAME \
    --description "Security group for Instance Public" \
    --vpc-id $VPC_ID \
    --query "GroupId" \
    --output text)
echo -e "\033[0;32mID SG PUBLIC\033[0m \033[1;31m$SG_ID\033[0m"
```
# OPCIONAL solo si se pierde la conexion
```bash
SG_ID=$(aws ec2 describe-security-groups \
    --filters "Name=vpc-id,Values=$VPC_ID" \
    --query "SecurityGroups[0].GroupId" \
    --output text)
echo -e "\033[0;32mID SG PUBLIC\033[0m \033[1;31m$SG_ID\033[0m"
```
### 5.2.1 Habilitar SSH
```bash
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```
# 5.3 Crear Instance
```bash
INSTANCE_ID=$(aws ec2 run-instances \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value='$INSTANCE_PUBLIC_NAME'}]" \
    --image-id $ID_IMAGE \
    --instance-type $INSTANCE_TYPE \
    --key-name $KEY_PAIR_NAME \
    --region $REGION \
    --security-group-ids $SG_ID \
    --subnet-id $SUBNET_ID \
    --query "Instances[0].InstanceId" \
    --output text)
echo -e "\033[0;32mID INSTANCE PUBLIC\033[0m \033[1;31m$INSTANCE_ID\033[0m"
```

```bash
aws ec2 wait instance-running \
    --instance-ids $INSTANCE_ID
```
# OPCIONAL solo si se pierde la conexion
```bash
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_PUBLIC_NAME" \
    --query "Reservations[0].Instances[0].InstanceId" \
    --output text)
echo -e "\033[0;32mID INSTANCE PUBLIC\033[0m \033[1;31m$INSTANCE_ID\033[0m"
```
# 6 Crear ROLE
```bash
cat << EOF > trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

```bash
aws iam create-role \
    --role-name $ROLE_NAME \
    --assume-role-policy-document file://trust-policy.json
```
## 6.1 Crear politica
```bash
cat << EOF > politica.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3: PutObject"
      ],
      "Resource": "arn:aws:s3:::$S3_NAME/*"
    }
  ]
}
EOF
```

```bash
POLITICA_ARN=$(aws iam create-policy \
    --policy-name politica_s3 \
    --policy-document file://politica.json \
    --region $REGION \
    --query "Policy.Arn" \
    --output text)
```
## 6.2 Asociar politica
```bash
aws iam attach-role-policy \
    --role-name $ROLE_NAME \
    --policy-arn $POLITICA_ARN \
    --region $REGION
```
# 7. Crear Instance Profile
Esto nos va permitir asignarle los permisos del rol a la instancia publica
```bash
aws iam create-instance-profile \
    --instance-profile-name profile-s3

aws iam add-role-to-instance-profile \
    --instance-profile-name profile-s3 \
    --role-name $ROLE_NAME
```
## 7.1 Asociar INSTANCE PROFILE con la INSTANCE Public
Debemos esperar unos segundos antes de ejecutar este comando.
```bash
aws ec2 associate-iam-instance-profile \
    --instance-id $INSTANCE_ID \
    --iam-instance-profile Name=profile-s3
```
# 8 Crear BUCKET S3 Private
```bash
aws s3api create-bucket \
    --acl private \
    --bucket $S3_NAME \
    --region $REGION
```
# 9 Crear Peering Connection
```bash
PEERING_ID=$(aws ec2 create-vpc-peering-connection \
    --vpc-id $VPC_ID \
    --peer-vpc-id $ID_VPC_PRIVATE \
    --tag-specifications "ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=$PEERING_NAME}]" \
    --query "VpcPeeringConnection.VpcPeeringConnectionId" \
    --output text)
echo -e "\033[0;32mID PERRING PUBLIC\033[0m \033[1;31m$PEERING_ID\033[0m"
```
#### Aceptar conexion
Con este comando vemos el estado de nuestra conexion.
```bash
aws ec2 accept-vpc-peering-connection \
    --vpc-peering-connection-id $PEERING_ID \
    --query "VpcPeeringConnection.Status.{Estado:Message}" \
    --output table
```
# OPCIONAL solo si se pierde la conexion
```bash
PEERING_ID=$(aws ec2 describe-vpc-peering-connections \
    --filters "Name=tag:Name,Values=$PEERING_NAME" \
    --query "VpcPeeringConnections[0].VpcPeeringConnectionId" \
    --output text)
echo -e "\033[0;32mID PERRING PUBLIC\033[0m \033[1;31m$PEERING_ID\033[0m"
```
# 9.1 Configurar tablas de rutas
#### 9.1.1 Ruta de la VPC publica a la VPC privada
Ya creamos la conexion, ahora lo que debemos crear es el camino para que esa conexion conosca por donde de ir desde la publica a la privada:
```bash
aws ec2 create-route \
    --route-table-id $RT_ID \
    --destination-cidr-block $VPC_CIDR_PRIVATE \
    --vpc-peering-connection-id $PEERING_ID
```

#### 9.1.2 Ruta de la VPC privada a la VPC publica
Ya creamos la conexion, ahora lo que debemos crear es el camino para que esa conexion conosca por donde de ir desde la privada a la publica:
```bash
aws ec2 create-route \
    --route-table-id $ID_RT_PRIVATE \
    --destination-cidr-block $VPC_CIDR_PUBLIC \
    --vpc-peering-connection-id $PEERING_ID
```
# 10 Obtener IP's de los servidores public y private
## 10.1 Obtener IP Publica
```bash
IP_PUBLIC=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_PUBLIC_NAME" \
    --query "Reservations[0].Instances[0].PublicIpAddress" \
    --output text)
echo -e "\033[0;32mIP PUBLIC\033[0m \033[1;31m$IP_PUBLIC\033[0m"
```
## 10.2 Obtener IP Privada INSTANCE PUBLIC
```bash
IP_PUBLIC_PRIVATE=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_PUBLIC_NAME" \
    --query "Reservations[0].Instances[0].PrivateIpAddress" \
    --output text)
echo -e "\033[0;32mIP PUBLIC PRIVATE\033[0m \033[1;31m$IP_PUBLIC_PRIVATE\033[0m"
```
## 10.2 Obtener IP Privada INSTANCE PRIVATE
```bash
IP_PRIVATE=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_PRIVATE_NAME" \
    --query "Reservations[0].Instances[0].PrivateIpAddress" \
    --output text)
echo -e "\033[0;32mIP PRIVATE\033[0m \033[1;31m$IP_PRIVATE\033[0m"
```
# 11 Copiar $KEY_PAIR_PATH
Vamos a copiar la llave .pem en el servidor publico para luego utilizarlo para conectarnos al servidor privado.
```bash
scp -i ~/.ssh/$PREFIX-key.pem $KEY_PAIR_PATH ubuntu@$IP_PUBLIC:/home/ubuntu/.ssh/
```
# 12 Ingresar a servidor publico
Abrir otra terminal
```bash
PREFIX=datacenter
IP_PUBLIC="204.236.247.206"
ssh -i ~/.ssh/$PREFIX-key.pem ubuntu@$IP_PUBLIC
```
#### Verificar conexion
```bash
IP_PRIVATE="10.10.1.50"
telnet $IP_PRIVATE 22
```
# 13 Ingresar al servidor privado desde el servidor publico
```bash
PREFIX=datacenter
IP_PRIVATE="10.10.1.50"
ssh -i ~/.ssh/$PREFIX-key.pem ubuntu@$IP_PRIVATE
```
## 13.1 Crear llaves
```bash
ssh-keygen -t rsa -b 4096
```

```bash
exit
```
Nos salimos del servidor privado para copiar la llave publica en el servidor publico
```bash
scp -i ~/.ssh/$PREFIX-key.pem ubuntu@$IP_PRIVATE:/home/ubuntu/.ssh/id_rsa.pub ~/.ssh/
```

```bash
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
```
Volvemos a ingresar al servidor privado
```bash
ssh -i ~/.ssh/$PREFIX-key.pem ubuntu@$IP_PRIVATE
```
# 14 Crear script para copiar logs
Vamos a copiar los logs del servidor privado al servidor publico
```bash
PREFIX=datacenter
IP_PUBLIC_PRIVATE="10.0.1.46"
cat << EOF > script_copy.sh
#!/bin/bash
#Filename: script_copy.sh
#Description: Copia los logs del servidor privado al servidor publico

scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i /home/ubuntu/.ssh/id_rsa /var/log/boots.log ubuntu@$IP_PUBLIC_PRIVATE:/home/ubuntu/boots.log
EOF
```

```bash
chmod 701 script_copy.sh
```
#### Crear crontab privado
```bash
crontab -e
########
##solo si los permisos de /var/log/boots.log son de root
sudo crontab -e
```

```bash
* * * * * /home/ubuntu/script_copy.sh >> /home/ubuntu/script_salida.log 2>&1
```

```bash
tail -f script_salida.log
```
# 15 Ingresar a servidor publico
Volvemos a ingresar al servidor publico, abrimos otra terminal
```bash
PREFIX=datacenter
IP_PUBLIC="204.236.247.206"
ssh -i ~/.ssh/$PREFIX-key.pem ubuntu@$IP_PUBLIC
```
#### 15.1 Actualizar repositorio de paquetes e instalamos awscli
```bash
sudo apt update
sudo apt install awscli -y
aws --version
```
#### 15.2 Login
Antes debemos obtener los Access Key ID y Secret Access Key en el host vamos a ejecutar: 
```bash
cat ~/.aws/credentials
```
En el servidor remoto ejecutamos
```bash
aws configure
```
#### 15.3 Crear script para subir logs a S3
```bash
PREFIX=datacenter
S3_NAME=$PREFIX-s3-logs-927609959
S3_PATH=$PREFIX-priv-vpc/boot/boots.log
cat << EOF > script_upload.sh
#!/bin/bash
#Filename: script_upload.sh
#Description: Sube los logs a S3

aws s3 cp /home/ubuntu/boots.log s3://$S3_NAME/$S3_PATH
EOF
```

```bash
chmod 701 script_upload.sh
```
#### Crear crontab
```bash
crontab -e
```

```bash
* * * * * /home/ubuntu/script_upload.sh >> /home/ubuntu/script_salida.log 2>&1
```

```bash
tail -f script_salida.log
```
# 16 Verificar
```bash
aws s3 ls s3://$S3_NAME/$S3_PATH
```

