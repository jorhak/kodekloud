```
The Nautilus DevOps team is tasked with enabling internet access for an EC2 instance running in a private subnet. This instance should be able to upload a test file to a public S3 bucket once it can access the internet. To minimize costs, the team has decided to use a NAT Instance instead of a NAT Gateway.

The following components already exist in the environment:  
1) A VPC named `xfusion-priv-vpc` and a private subnet named `xfusion-priv-subnet` have been created.  
2) An EC2 instance named `xfusion-priv-ec2` is already running in the private subnet.  
3) The EC2 instance is configured with a cron job that uploads a test file to the S3 bucket `xfusion-nat-25028` every minute. Upload will only succeed once internet access is established.

Your task is to:

- Create a new public subnet named `xfusion-pub-subnet` in the existing VPC.
- Launch a NAT Instance in the public subnet using an Amazon Linux 2023 AMI and name it `xfusion-nat-instance`. Configure this instance to act as a NAT instance. Make sure to use a custom security group for this instance.

After the configuration, verify that the test file `xfusion-test.txt` appears in the S3 bucket `xfusion-nat-25028`. This indicates successful internet access from the private EC2 instance via the NAT Instance.

Note: `iptables` is not installed by default on Amazon Linux 2023. You will need to install and enable it before configuring NAT setup.

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://062665041731.signin.aws.amazon.com/console?region=us-east-1]|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Mon May 18 15:12:22 UTC 2026|
|End Time|Mon May 18 15:12:22 UTC 2026|

  
`Notes:`

- Use region `us-east-1`
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```
# Variables de entorno
```
PREFIX=nautilus
VPC_PRIVATE_NAME=$PREFIX-priv-vpc
SUBNET_PRIVATE_NAME=$PREFIX-priv-subnet
INSTANCE_PRIVATE_NAME=$PREFIX-priv-ec2
S3_NAME=$PREFIX-nat-1665
SUBNET_PUBLIC_NAME=$PREFIX-pub-subnet
INSTANCE_NAT_NAME=$PREFIX-nat-instance
INSTANCE_TYPE=t3.micro
REGION=us-east-1
SG_DESCRIPTION="sg para la instancia nat"
SG_NAME=$PREFIX-sg-nat-instance
RT_PUBLIC_NAME=$PREFIX-ruta-publica
RT_PRIVATE_NAME=$PREFIX-ruta-privada
IGW_NAME=$PREFIX-igw
KEY_NAME=mi-llave-publica
```
# Buscar la imagen Amazon Linux 2023

```
aws ssm get-parameters-by-path \
    --path /aws/service/ami-amazon-linux-latest \
    --query "Parameters[].Name"
```

```
IMAGE_ID=$(aws ssm get-parameter \
    --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64 \
    --region $REGION \
    --query "Parameter.Value" \
    --output text)
```

El atributo --name debe ser reemplazado por el valor de la lista del comando previo.
# 1 Verificar VPC Private

```
aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=$VPC_PRIVATE_NAME \
    --query "Vpcs[0].{CIDR:CidrBlockAssociationSet[0].CidrBlock,NOMBRE:Tags[?Key=='Name'].Value | [0],ESTADO:State}" \
    --output table
```
#### CIDR de VPC Private
```
VPC_CIDR=$(aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=$VPC_PRIVATE_NAME \
    --query "Vpcs[0].CidrBlockAssociationSet[0].CidrBlock" \
    --output text)
```
#### ID de VPC PRIVATE
```
VPC_ID=$(aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=$VPC_PRIVATE_NAME \
    --region $REGION \
    --query "Vpcs[0].VpcId" \
    --output text)
```

# 2 Verificar Subnet Private
```
aws ec2 describe-subnets \
    --filters Name=tag:Name,Values=$SUBNET_PRIVATE_NAME \
    --query "Subnets[0].{NOMBRE:Tags[?Key=='Name'].Value | [0],ESTADO:State,CIDR:CidrBlock}" \
    --output table
```
#### ID de Subnet Private
```
SUBNET_PRIVATE_ID=$(aws ec2 describe-subnets \
    --filters Name=tag:Name,Values=$SUBNET_PRIVATE_NAME \
    --region $REGION \
    --query "Subnets[0].SubnetId" \
    --output text)
```
# 3 Verificar si la VPC cuenta con Internet Gateway
```
aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=$VPC_ID" \
    --query "InternetGateways[0].{ID_IGW:InternetGatewayId}" \
    --output table
```
# 4 Crear y adjuntar Internet Gateway a la VPC Private
#### Crear Internet Gateway
```
IGW_ID=$(aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=$IGW_NAME}]" \
    --region $REGION \
    --query 'InternetGateway.InternetGatewayId' \
    --output text)
```
#### Adjuntar IGW a VPC Private
```
aws ec2 attach-internet-gateway \
    --vpc-id $VPC_ID \
    --internet-gateway-id $IGW_ID \
    --region $REGION
```
**Volvemos a ejecutar el comando del paso tres.**
# 5 Crear SUBNET PUBLIC
```
AZ=$(aws ec2 describe-subnets \
--region $REGION \
--subnet-ids $SUBNET_PRIVATE_ID \
--query "Subnets[0].AvailabilityZone" \
--output text)
```

```
SUBNET_PUBLIC_ID=$(aws ec2 create-subnet \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=$SUBNET_PUBLIC_NAME}]" \
    --vpc-id $VPC_ID \
    --cidr-block 10.1.2.0/24 \
    --availability-zone $AZ \
    --region $REGION \
    --query "Subnet.SubnetId" \
    --output text)
```
#### Habilitar asignacion IP Publica
```
aws ec2 modify-subnet-attribute \
    --subnet-id $SUBNET_PUBLIC_ID \
    --map-public-ip-on-launch \
    --region $REGION
```

# 5 Crear ROUTE TABLE PUBLIC y asociarla a VPC Private
```
RT_PUBLIC_ID=$(aws ec2 create-route-table \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=$RT_PUBLIC_NAME}]" \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "RouteTable.RouteTableId" \
    --output text)
```
#### Crear ruta hacia Internet
```
aws ec2 create-route \
    --route-table-id $RT_PUBLIC_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID \
    --region $REGION
```
#### Asociar Route Table a SUBNET PUBLIC
```
aws ec2 associate-route-table \
    --subnet-id $SUBNET_PUBLIC_ID \
    --route-table-id $RT_PUBLIC_ID \
    --region $REGION
```
# 7 Crear Security Group para la instancia NAT
#### Security Group
```
SG_NAT_ID=$(aws ec2 create-security-group \
    --description "$SG_DESCRIPTION" \
    --group-name $SG_NAME \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "GroupId" \
    --output text)
```
#### Habilitar puerto SSH
```
aws ec2 authorize-security-group-ingress \
--region $REGION \
--group-id $SG_NAT_ID \
--protocol tcp \
--port 22 \
--cidr 0.0.0.0/0
```
#### Permitir conexion en la VPC
```
aws ec2 authorize-security-group-ingress \
--region $REGION \
--group-id $SG_NAT_ID \
--protocol -1 \
--cidr $VPC_CIDR
```
# 8 Crear y configurar VM NAT
#### Obtener IP Private
Obtenemos la IP privada de la instancia que esta en la subnet privada.
```
aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_PRIVATE_NAME \
    --query "Reservations[].Instances[].{Nombre:Tags[?Key=='Name'].Value |[0],Estado:State.Name,Tipo:InstanceType,IPPrivada:PrivateIpAddress}" \
    --output table
```
#### Obtener Subnet Private
Obtenemos la Subnet privada para comparala con la de nuestra IP Private que debe estar en el rango.
```
aws ec2 describe-subnets \
    --filters Name=tag:Name,Values=$SUBNET_PRIVATE_NAME \
    --query "Subnets[0].{NOMBRE:Tags[?Key=='Name'].Value | [0],ESTADO:State,CIDR:CidrBlock}" \
    --output table
```
#### Script de instalacion
Con la comprobacion que hicimos previamente, en nuestro script vamos a colocar el CIDR de la Subnet Private.
```
vi install.sh
```

```
#!/bin/bash
#Filename: install.sh
#Description: Configuracion para servidor Nat
IP_PRIVATE=10.1.1.0/24
dnf update -y
dnf install -y iptables-services
dnf install -y conntrack-tools
echo "net.ipv4.ip_forward = 1" > /etc/sysctl.d/99-nat-forwarding.conf
sysctl -p /etc/sysctl.d/99-nat-forwarding.conf
systemctl start iptables
systemctl enable iptables
PRIMARY_INTERFACE=$(ip route | grep default | awk '{print $5}')
iptables -t nat -A POSTROUTING -s $IP_PRIVATE -o $PRIMARY_INTERFACE -j MASQUERADE
iptables -I FORWARD 1 -s $IP_PRIVATE -j ACCEPT
iptables -I FORWARD 2 -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -j ACCEPT
service iptables save
systemctl restart iptables
```
#### Crear par de llaves
```
ssh-keygen -t rsa -b 4096
```
#### Importar Key Pair
```
aws ec2 import-key-pair \
    --key-name $KEY_NAME \
    --public-key-material fileb:///root/.ssh/id_rsa.pub
```
#### Crear instancia
```
INSTANCE_NAT_ID=$(aws ec2 run-instances \
    --image-id $IMAGE_ID \
    --instance-type $INSTANCE_TYPE \
    --security-group-ids $SG_NAT_ID \
    --subnet-id $SUBNET_PUBLIC_ID \
    --user-data file://install.sh \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$INSTANCE_NAT_NAME}]" \
    --region $REGION \
    --key-name $KEY_NAME \
    --query "Instances[0].InstanceId" \
    --output text)
```

```
aws ec2 wait instance-running \
    --instance-ids $INSTANCE_NAT_ID \
    --region $REGION
```

#### (Opcional) Otra forma de capturar ID de la instancia
```
INSTANCE_NAT_ID=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAT_NAME \
    --query "Reservations[].Instances[].InstanceId" \
    --output text)
```
#### Deshabilitar la verificacion de Origin/Destino
```
aws ec2 modify-instance-attribute \
    --instance-id $INSTANCE_NAT_ID \
    --no-source-dest-check \
    --region $REGION
```
# 9 Enrutar la SUBNET PRIVADA hacia la instancia NAT
#### Obtener ID de ROUTE TABLE de SUBNET PRIVATE
```
RT_PRIVATE_ID=$(aws ec2 describe-route-tables \
    --filters "Name=association.subnet-id,Values=$SUBNET_PRIVATE_ID" \
    --query "RouteTables[0].RouteTableId" \
    --output text)
```
#### Crear ruta hacia NAT
Antes de ejecutar este comando abrimos otra terminal, inicializamos las variables de entorno, caputramos el INSTANCE_NAT_ID y luego pasamos a paso 10.
```
aws ec2 create-route \
    --route-table-id $RT_PRIVATE_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --instance-id $INSTANCE_NAT_ID \
    --region $REGION
```
# 10 Ingresar al servidor NAT
#### Obtener IP Publica
```
IP_PUBLIC=$(aws ec2 describe-instances \
    --instance-ids $INSTANCE_NAT_ID \
    --query "Reservations[0].Instances[0].PublicIpAddress" \
    --output text)
USER=ec2-user
```
#### Ingresar a la instancia NAT
```
ssh $USER@$IP_PUBLIC
```

# 11 Verificar
Abrimos otra terminal, inicializamos las variables de entorno
```
aws s3 ls s3://$S3_NAME/ --region $REGION
```
Volvemos a la terminal donde ingresamos al servidor NAT.
Este comando nos permite ver los permisos que le hemos dado al servidor NAT
```
sudo iptables -L FORWARD -n --line-numbers
```
Este comando nos permite ver la conexion entre el servidor privado y internet
```
sudo conntrack -L --src <IP_PRIVATE>
```
Nos permite ver la entrada y salida que tiene nuestro servidor privado a traves del servidor NAT.
```
sudo tcpdump -n -i any host <IP_PRIVATE>
```

Ejecutamos el comando que dejamos a la espera en el paso 9 **Crear ruta hacia NAT** Vamos a ver que el trafico ya esta permitido.