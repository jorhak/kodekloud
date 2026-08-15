```
The Nautilus Development Team recently deployed a new web application hosted on an EC2 instance within a public VPC named xfusion-vpc. The application, running on an Nginx server, should be accessible from the internet on port 80. Despite configuring the security group xfusion-sg to allow traffic on port 80 and verifying the EC2 instance settings, the application remains inaccessible from the internet. The team suspects that the issue might be related to the VPC configuration, as all other components appear to be set up correctly. The DevOps team has been asked to troubleshoot and resolve the issue to ensure the application is accessible to external users.

As a member of the Nautilus DevOps Team, your task is to perform the following:

Verify VPC Configuration: Ensure that the VPC xfusion-vpc is properly configured to allow internet access.

Ensure Accessibility: Make sure the EC2 instance xfusion-ec2 running the Nginx server is accessible from the internet on port 80.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://861007547049.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Fri Aug 14 17:13:04 UTC 2026
End Time	Fri Aug 14 18:13:04 UTC 2026

Notes:

Create the resources only in us-east-1 region.
To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:\ntoggle button
```
# Variables de entorno
```
PREFIX=devops
VPC_NAME=$PREFIX-vpc
SG_NAME=$PREFIX-sg
INSTANCE_NAME=$PREFIX-ec2
IGW_NAME=$PREFIX-igw
REGION=us-east-1
```
# Obtener VPC ID
```
VPC_ID=$(aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=$VPC_NAME \
    --query "Vpcs[0].VpcId" \
    --output text)
echo -e "\033[0;32mMi VPC::\033[0m \033[1;31m${VPC_ID}\033[0m"
```
# Obtener Route Table ID
```
RT_ID=$(aws ec2 describe-route-tables \
    --filters Name=vpc-id,Values=$VPC_ID \
    --query "RouteTables[1].RouteTableId" \
    --output text)
echo -e "\033[0;32mMi Route Table::\033[0m \033[1;31m${RT_ID}\033[0m"
```
Debemos tener cuidado cuando utilizamos **[0]** puede que no este donde nosotros esperamos sino que se encuentre en **[1]**.
# Obtener Subnet de VPC
```
SUBNET_ID=$(aws ec2 describe-subnets \
    --filters Name=vpc-id,Values=$VPC_ID \
    --region $REGION \
    --query "Subnets[0].SubnetId" \
    --output text)
echo -e "\033[0;32mMi subnet::\033[0m \033[1;31m${SUBNET_ID}\033[0m"
```

# 1 Verificar que la VPC tenga Internet Gateway
```
aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=$VPC_ID" \
    --query "InternetGateways[0].{ID_IGW:InternetGatewayId}" \
    --output table
```
Vemos que nuestra VPC no cuenta con una IGW.
## 2 Crear y adjuntar Internet Gateway
## 2.1 Crear Internet Gateway
```
IGW_ID=$(aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=$IGW_NAME}]" \
    --region $REGION \
    --query 'InternetGateway.InternetGatewayId' \
    --output text)
echo -e "\033[0;32mMi IGW::\033[0m \033[1;31m${IGW_ID}\033[0m"
```
## 2.2 Adjuntar Internet Gateway a VPC
```
aws ec2 attach-internet-gateway \
    --vpc-id $VPC_ID \
    --internet-gateway-id $IGW_ID \
    --region $REGION
```
Volvemos a ejecutar el comando del paso 1.
Ahora nuestra VPC cuenta con una IGW.
# 3 Verificar que nuestro Route Table tenga nuestra IGW
```
aws ec2 describe-route-tables \
    --filters Name=association.gateway-id,Values=$IGW_ID 
```
Vemos que ninguna Route Table tiene nuestra IGW.
## 3.1 Verificar que rutas tiene nuestro RT
```
aws ec2 describe-route-tables \
    --filters Name=route-table-id,Values=$RT_ID \
    --query "RouteTables[0].Routes[*].{Destino:DestinationCidrBlock,GatewayID:GatewayId,Origen:Origin,Estado:State}" \
    --output table
```
Podemos ver que tenemos una IGW que no va a ningun lado, su estado es **backhole** y que no es nuestra IGW.
## 3.2 Crear ruta
```
aws ec2 create-route \
    --route-table-id $RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID \
    --region $REGION
```
Nos dice que la ruta ya existe y tiene razon lo acabamos de ver pero su IGW no va a ningun lado, lo que debemos hacer es reemplazarla el IGW que tiene por el que creamos. 
## 3.3 Reemplazar el IGW actual por el que creamos
```
aws ec2 replace-route \
    --route-table-id $RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID
```
Volvemos a ejecutar el comando del paso 3.1. Y podemos ver que el estado esta activo.
# 4 Verificamos que Security Group tenga el permiso
```
aws ec2 describe-security-groups \
    --filters Name=vpc-id,Values=$VPC_ID Name=group-name,Values=$SG_NAME
```
## 4.1 Obtener Security Group ID
```
SG_ID=$(aws ec2 describe-security-groups \
    --filters Name=group-name,Values=$SG_NAME Name=group-name,Values=$SG_NAME \
    --query "SecurityGroups[0].GroupId" \
    --output text)
```
## 4.2 Agregar permiso SSH
```
aws ec2 authorize-security-group-ingress \
    --region $REGION \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```
Ejecutamos el comando del paso 4 para ver que se agrego el permiso
# 4 Ver Instance
```
aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[0].Instances[].{Nombre:Tags[?Key=='Name'].Value | [0],Estado:State.Name,IpPublica:PublicIpAddress}" \
    --output table
```
## 4.1 Obtener usuario y IP publica
```
IP_PUBLIC=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[0].Instances[].PublicIpAddress" \
    --output text)
USER=ec2-user
```
## 4.2 Crear Key pair
```
ssh-keygen -t rsa -b 4096
```
### 4.2.1 Copiamos la llave publica
```
cat .ssh/id_rsa.pub
```
## 4.3 Ingresar a la consola de AWS
**EC2>Instances>** marcamos **$INSTANCE_NAME** damos click en Connect seleccionamos la pestana **In web browser** y elegimos **EC2 Instance Connect**, en **Settings** lo dejamos en **IPv4** finalmente damos click en Connect.

Se nos va abrir una terminal y lo que debemos hacer es copiar la llave publica en authorized_keys de ambos usuarios: ec2-user y root.
```
vi ~/.ssh/authorized_keys
sudo su
vi ~/.ssh/authorized_keys
```
# 5 Ingresar a servidor
```
ssh $USER@$IP_PUBLIC
```
Verificamos que **Nginx** instalado y corriendo.
```
sudo systemctl status nginx
sudo nginx -t
exit
```
# 6 Verificar
### 6.1 Verificar a traves de curl
```
curl -i http://$IP_PUBLIC
```
### 6.2 Verificar por el navegador
```
echo "http://${IP_PUBLIC}"
```


