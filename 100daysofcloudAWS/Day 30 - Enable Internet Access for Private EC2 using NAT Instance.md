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
VPC_PRIVATE_NAME=xfusion-priv-vpc
SUBNET_PRIVATE_NAME=xfusion-priv-subnet
INSTANCE_PRIVATE_NAME=xfusion-priv-ec2
S3_NAME=xfusion-nat-19128
SUBNET_PUBLIC_NAME=xfusion-pub-subnet
INSTANCE_NAT_NAME=xfusion-nat-instance
INSTANCE_TYPE=t2.micro
REGION=us-east-1
SG_DESCRIPTION="sg para la instancia nat"
SG_NAME=nautilus-sg-nat-instance
RT_PUBLIC_NAME=xfusion-ruta-publica
RT_PRIVATE_NAME=xfusion-ruta-privada
IGW_NAME=devops-igw
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

```
echo $IMAGE_ID
```
El atributo --name debe ser reemplazado por el valor de la lista del comando previo.
# 1 Obtener ID, CIDR de VPC y CIDR de SUBNET PRIVATE
El **CIDR** para evitar colisiones, y el VPC_ID para asociarlo a los objetos que se van a crear mas adelante.
#### Obtener id de VPC
```
VPC_ID=$(aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=$VPC_PRIVATE_NAME \
    --region $REGION \
    --query "Vpcs[0].VpcId" \
    --output text)
```

```
echo $VPC_ID
```
#### Obtener cidr de VPC
```
VPC_CIDR=$(aws ec2 describe-vpcs \
--region $REGION \
--vpc-ids $VPC_ID \
--query "Vpcs[0].CidrBlock" \
--output text)
```

```
echo $VPC_CIDR
```
#### Obtener ID de SUBNET PRIVATE
```
SUBNET_PRIVATE_ID=$(aws ec2 describe-subnets \
    --filters "Name=tag:Name,Values=$SUBNET_PRIVATE_NAME" \
    --region $REGION \
    --query "Subnets[0].SubnetId" \
    --output text)
```

```
echo $SUBNET_PRIVATE_ID
```
#### Obtener cidr de SUBNET PRIVATE
```
SUBNET_PRIVATE_CIDR=$(aws ec2 describe-subnets \
    --filters "Name=tag:Name,Values=$SUBNET_PRIVATE_NAME" \
    --region $REGION \
    --query "Subnets[0].CidrBlock" \
    --output text)
```

```
echo $SUBNET_PRIVATE_CIDR
```
#### Verificar si tiene VPC cuenta con Internet Gateway
```
aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=$VPC_ID"
```
Podemos ver que no asignada una Internet Gateway.
# 2 Crear y adjuntar INTERNET GATEWAY (IGW)
```
IGW_ID=$(aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=$IGW_NAME}]" \
    --region $REGION \
    --query 'InternetGateway.InternetGatewayId' \
    --output text)
```

```
echo $IGW_ID
```
#### Adjuntar IGW a VPC
```
aws ec2 attach-internet-gateway \
    --vpc-id $VPC_ID \
    --internet-gateway-id $IGW_ID \
    --region $REGION
```
#### Verificar que nuestra VPC PRIVATE esta asociada a IGW
```
aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=$VPC_ID" \
    --query "InternetGateways[0].{ID_IGW:InternetGatewayId}" \
    --output table
```
# 3 Crear SUBNET PUBLIC
```
AZ=$(aws ec2 describe-subnets \
--region $REGION \
--subnet-ids $SUBNET_PRIVATE_ID \
--query "Subnets[0].AvailabilityZone" \
--output text)
```

```
echo $AZ
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

```
echo $SUBNET_PUBLIC_ID
```
# 4 Habilitar asignacion de IP Publica
```
aws ec2 modify-subnet-attribute \
    --subnet-id $SUBNET_PUBLIC_ID \
    --map-public-ip-on-launch \
    --region $REGION
```

# 5 Crear ROUTE TABLE PUBLIC y asociarla
```
RT_PUBLIC_ID=$(aws ec2 create-route-table \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=$RT_PUBLIC_NAME}]" \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "RouteTable.RouteTableId" \
    --output text)
```

```
echo $RT_PUBLIC_ID
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
# 6 Crear Security Group de la instancia NAT
```
SG_NAT_ID=$(aws ec2 create-security-group \
    --description "$SG_DESCRIPTION" \
    --group-name $SG_NAME \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "GroupId" \
    --output text)
```

```
aws ec2 authorize-security-group-ingress \
--region $REGION \
--group-id $SG_NAT_ID \
--protocol tcp \
--port 22 \
--cidr 0.0.0.0/0
```

```
aws ec2 authorize-security-group-ingress \
--region $REGION \
--group-id $SG_NAT_ID \
--protocol -1 \
--cidr $VPC_CIDR
```
# 7 Crear y configurar instancia NAT
#### Crear iptable.sh
```
vi iptable.sh
```

```
#!/bin/bash
dnf update -y
dnf install -y iptables-services
echo "net.ipv4.ip_forward = 1" > /etc/sysctl.d/99-nat-forwarding.conf
sysctl -p /etc/sysctl.d/99-nat-forwarding.conf
systemctl start iptables
systemctl enable iptables
PRIMARY_INTERFACE=$(ip route | grep default | awk '{print $5}')
iptables -t nat -A POSTROUTING -o $PRIMARY_INTERFACE -j MASQUERADE
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
    --user-data file://iptable.sh \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$INSTANCE_NAT_NAME}]" \
    --region $REGION \
    --key-name $KEY_NAME \
    --query "Instances[0].InstanceId" \
    --output text)
```

```
aws ec2 wait instance-running --instance-ids $INSTANCE_NAT_ID --region $REGION
```
#### Deshabilitar la verificacion de Origin/Destino
```
aws ec2 modify-instance-attribute \
    --instance-id $INSTANCE_NAT_ID \
    --no-source-dest-check \
    --region $REGION
```
# 9 Enrutar la SUBNET PRIVADA hacia la instancia NAT
#### Obtener ID de ROUTE TABLE PRIVATE
```
RT_PRIVATE_ID=$(aws ec2 describe-route-tables \
    --filters "Name=association.subnet-id,Values=$SUBNET_PRIVATE_ID" \
    --query "RouteTables[0].RouteTableId" \
    --output text)
```

```
echo $RT_PRIVATE_ID
```
#### Crear ruta hacia la instancia NAT
```
aws ec2 create-route \
    --route-table-id $RT_PRIVATE_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --instance-id $INSTANCE_NAT_ID \
    --region $REGION
```
# Verificar
```
IP_PUBLIC=$(aws ec2 describe-instances \
    --instance-ids $INSTANCE_NAT_ID \
    --query "Reservations[0].Instances[0].PublicIpAddress" \
    --output text)
```

```
USER=ec2-user
```

```
ssh $USER@$IP_PUBLIC
```

```
aws s3 ls s3://$S3_NAME/ --region $REGION
```

```
aws s3 cp iptable.sh s3://$S3_NAME/iptables.sh 
```


```
aws ec2 terminate-instances \
    --instance-ids $INSTANCE_NAT_ID
```

```
aws ec2 wait instance-terminated \
    --instance-ids $INSTANCE_NAT_ID
```

```
aws ec2 describe-route-tables \
    --route-table-ids $RT_PRIVATE_ID
```

```
IP_PRIVATE=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_PRIVATE_NAME \
    --query "Reservations[0].Instances[0].{IpPublica:PublicIpAddress,IpPrivada:PrivateIpAddress,Estado:State.Name,Nombre:Tags[0].Value}" \
    --output table)
```

```
ssh user-ec2@$IP_PRIVATE
```