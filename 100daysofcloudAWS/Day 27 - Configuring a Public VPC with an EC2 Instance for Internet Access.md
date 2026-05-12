```
The Nautilus DevOps Team has received a request from the Networking Team to set up a new public VPC to support a set of public-facing services. This VPC will host various resources that need to be accessible over the internet. As part of this setup, you need to ensure the VPC has public subnets with automatic IP assignment for resources. Additionally, a new EC2 instance will be launched within this VPC to host public applications that require SSH access. This setup will enable the Networking Team to deploy and manage public-facing applications.

Create a public VPC named nautilus-pub-vpc, and a subnet named nautilus-pub-subnet under the same, make sure public IP is being auto assigned to resources under this subnet. Further, create an EC2 instance named nautilus-pub-ec2 under this VPC with instance type t2.micro. Make sure SSH port 22 is open for this instance and accessible over the internet.


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://982573538438.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Wed May 06 18:11:42 UTC 2026
End Time	Wed May 06 19:11:42 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

# Variables de entorno
```
VPC_NAME=datacenter-pub-vpc
SUBNET_NAME=datacenter-pub-subnet
INSTANCE_NAME=datacenter-pub-ec2
TYPE=t2.micro
PORT_SSH=22
IMAGE_ID=ami-0b6c6ebed2801a5cb
TYPE=t2.micro
REGION=us-east-1
KEY_NAME=millave
SG_NAME=datacenter-security-group
SG_DESCRIPTION="Security Group para una VPC publica"
IGW_NAME="datacenter-internet-gateway"
RT_NAME="datacenter-route-table"
```

# 1 Crear VPC
```
VPC_ID=$(aws ec2 create-vpc \
--cidr-block 10.0.0.0/16 \
--tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value='$VPC_NAME'}]" \
--region $REGION \
--query "Vpc.VpcId" \
--output text)
```

# 2 Crear Internet Gateway (IGW)
### Crear IGW
```
IGW_ID=$(aws ec2 create-internet-gateway \
--tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value='$IGW_NAME'}]" \
--region $REGION \
--query "InternetGateway.InternetGatewayId" \
--output text)
```
### Asociar IGW con VPC
```
aws ec2 attach-internet-gateway \
    --internet-gateway-id $IGW_ID \
    --vpc-id $VPC_ID \
    --region $REGION
```
# 3 Crear Subnet
```
SUBNET_ID=$(aws ec2 create-subnet \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value='$SUBNET_NAME'}]" \
    --cidr-block 10.0.1.0/24 \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "Subnet.SubnetId" \
    --output text)
```
### Esperar a que se cree la subnet
```
aws ec2 wait subnet-available \
    --subnet-ids $SUBNET_ID
```
### Habilitar IP Publica
```
aws ec2 modify-subnet-attribute \
    --map-public-ip-on-launch \
    --subnet-id $SUBNET_ID \
    --region $REGION
```
# 4 Enrutamiento
### Crear tabla de rutas
```
RT_ID=$(aws ec2 create-route-table \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value='$RT_NAME'}]" \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "RouteTable.RouteTableId" \
    --output text)
```
### Crear ruta hacia el IGW
```
aws ec2 create-route \
    --route-table-id $RT_ID \
    --destination-cidr-bloc 0.0.0.0/0 \
    --gateway-id $IGW_ID \
    --region $REGION
```
### Asignar la tabla a nuestra subred
```
aws ec2 associate-route-table \
    --subnet-id $SUBNET_ID \
    --route-table-id $RT_ID \
    --region $REGION
```
# 5 Crear Security Group
```
SG_ID=$(aws ec2 create-security-group \
    --description "$SG_DESCRIPTION" \
    --group-name $SG_NAME \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "GroupId" \
    --output text)
```
### Habilitar puerto
```
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port $PORT_SSH \
    --cidr 0.0.0.0/0 \
    --region $REGION
```
# 6 Crear Instancia
### Par de llaves
```
aws ec2 create-key-pair \
    --key-name $KEY_NAME \
    --key-type rsa \
    --key-format pem \
    --region $REGION \
    --query "KeyMaterial" \
    --output text > millave.pem
```

```
chmod 400 millave.pem
```
### Instancia
```
INSTANCE_ID=$(aws ec2 run-instances \
    --image-id $IMAGE_ID \
    --instance-type $TYPE \
    --security-group-ids $SG_ID \
    --subnet-id $SUBNET_ID \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value='$INSTANCE_NAME'}]" \
    --count 1 \
    --region $REGION \
    --key-name $KEY_NAME \
    --query "Instances[].InstanceId" \
    --output text)
```

```
aws ec2 wait instance-running \
    --instance-ids $INSTANCE_ID \
    --region $REGION
```
# 7 Estado de instancia
```
aws ec2 describe-instances \
    --instance-ids $INSTANCE_ID \
    --query "Reservations[].Instances[].{ID:InstanceId,IpPublica:PublicIpAddress,IpPrivada:PrivateIpAddress,State:State.Name,Nombre:Tags[?Key=='Name'].Value}" \
    --output table
```
### Conectar
```
ssh -i millave.pem ubuntu@<IpPublica>
```