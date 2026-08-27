```
The Nautilus DevOps team is tasked with enabling internet access for an EC2 instance running in a private subnet. This instance should be able to upload a test file to a public S3 bucket once it can access the internet. To achieve this, the team must set up a NAT Gateway in a public subnet within the same VPC.

1) A VPC named devops-priv-vpc and a private subnet devops-priv-subnet have already been created.
2) An EC2 instance named devops-priv-ec2 is already running in the private subnet.
3) The EC2 instance is configured with a cron job that uploads a test file to a bucket devops-nat-401749606 once internet is accessible.

Your task is to:

Create a public subnet named devops-pub-subnet in the same VPC.
Create an Internet Gateway and attach it to the VPC.
Create a route table devops-pub-rt and associate it with the public subnet.
Allocate an Elastic IP and create a NAT Gateway named devops-natgw.
Create a new, dedicated private route table named devops-priv-rt (do not reuse or edit the VPC's Main route table), explicitly associate it with the private subnet devops-priv-subnet, and add a route for 0.0.0.0/0 via the NAT Gateway.
Once complete, verify that the EC2 instance can reach the internet by confirming the presence of the test file in the S3 bucket devops-nat-401749606. After completing all the configuration, please wait a few minutes for the test file to appear in the bucket, as it may take 2–3 minutes.


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://683588789756.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Wed Aug 26 13:02:23 UTC 2026
End Time	Wed Aug 26 13:02:23 UTC 2026

Notes:

Use region us-east-1
To show/hide terminal: use the panel toggle button.
```
# Variables de entorno
```
PREFIX=xfusion
VPC_PRIVATE_NAME=$PREFIX-priv-vpc
SUBNET_PRIVATE_NAME=$PREFIX-priv-subnet
INSTANCE_PRIVATE_NAME=$PREFIX-priv-ec2
S3_NAME=$PREFIX-nat-589375092
SUBNET_PUBLIC_NAME=$PREFIX-pub-subnet
IGW_NAME=$PREFIX-igw
RT_PUBLIC_NAME=$PREFIX-pub-rt
NATGW_NAME=$PREFIX-natgw
RT_PRIVATE_NAME=$PREFIX-priv-rt
REGION=us-east-1
```
# Obtener ID de los recursos existentes privados
## Obtener VPC ID
```bash
VPC_PRIVATE_ID=$(aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=$VPC_PRIVATE_NAME \
    --query "Vpcs[0].VpcId" \
    --output text)
echo -e "\033[0;32mMi Vpc private::\033[0m \033[1;31m${VPC_PRIVATE_ID}\033[0m"
```
## Obtener SUBNET ID
```bash
SUBNET_PRIVATE_ID=$(aws ec2 describe-subnets \
    --filters Name=tag:Name,Values=$SUBNET_PRIVATE_NAME \
    --query "Subnets[0].SubnetId" \
    --output text)
echo -e "\033[0;32mMi Subnet private::\033[0m \033[1;31m${SUBNET_PRIVATE_ID}\033[0m"
```
## Obtener INSTANCE ID
```bash
INSTANCE_PRIVATE_ID=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_PRIVATE_NAME \
    --query "Reservations[0].Instances[0].InstanceId" \
    --output text)
echo -e "\033[0;32mMi Instance private::\033[0m \033[1;31m${INSTANCE_PRIVATE_ID}\033[0m"
```
# 1 Crear Subnet Publica en la VPC
```bash
SUBNET_PUBLIC_ID=$(aws ec2 create-subnet \
    --vpc-id $VPC_PRIVATE_ID \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=$SUBNET_PUBLIC_NAME},{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --cidr-block 10.1.30.0/24 \
    --region $REGION \
    --query "Subnet.SubnetId" \
    --output text)
echo -e "\033[0;32mMi Subnet publica::\033[0m \033[1;31m${SUBNET_PUBLIC_ID}\033[0m"
```
## 1.1 Habilitar asignacion de IP Publica
```bash
aws ec2 modify-subnet-attribute \
    --map-public-ip-on-launch \
    --subnet-id $SUBNET_PUBLIC_ID \
    --region $REGION
```
# 2 Crear Internet Gateway
```bash
IGW_ID=$(aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=$IGW_NAME},{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --region $REGION \
    --query 'InternetGateway.InternetGatewayId' \
    --output text)
echo -e "\033[0;32mMi IGW::\033[0m \033[1;31m${IGW_ID}\033[0m"
```
## 2.1 Adjuntar IGW a VPC Private
```bash
aws ec2 attach-internet-gateway \
    --vpc-id $VPC_PRIVATE_ID \
    --internet-gateway-id $IGW_ID \
    --region $REGION
```
## 2.2 Verificar que se adjunto
```bash
aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=$VPC_PRIVATE_ID" \
    --query "InternetGateways[0].{ID_IGW:InternetGatewayId}" \
    --output table
```
# 3 Crear Route Table publica y asociarla con Subnet publica
## 3.1 Crear Route Table
```bash
RT_PUBLIC_ID=$(aws ec2 create-route-table \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=$RT_PUBLIC_NAME},{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --vpc-id $VPC_PRIVATE_ID \
    --region $REGION \
    --query "RouteTable.RouteTableId" \
    --output text)
echo -e "\033[0;32mMi RT PUBLIC::\033[0m \033[1;31m${RT_PUBLIC_ID}\033[0m"
```
## 3.2 Crear ruta a IWG
```bash
aws ec2 create-route \
    --route-table-id $RT_PUBLIC_ID \
    --destination-cidr-bloc 0.0.0.0/0 \
    --gateway-id $IGW_ID \
    --region $REGION
```
## 3.2 Adjuntar Subnet publica a Route Table
```bash
aws ec2 associate-route-table \
    --subnet-id $SUBNET_PUBLIC_ID \
    --route-table-id $RT_PUBLIC_ID \
    --region $REGION
```
# 4 Crear Nat Gateway
## 4.1 Solicitar Elastic IP
```bash
ALLOCATION_ID=$(aws ec2 allocate-address \
    --domain vpc \
    --tag-specifications "ResourceType=elastic-ip,Tags=[{Key=Name,Value=$NATGW_NAME-eip},{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --query "AllocationId" \
    --output text
    --region $REGION)
echo -e "\033[0;32mMi Elastic IP::\033[0m \033[1;31m${ALLOCATION_ID}\033[0m"
```
## 4.2 Crear Nat Gateway
```bash
NATGW_ID=$(aws ec2 create-nat-gateway \
    --tag-specifications "ResourceType=natgateway,Tags=[{Key=Name,Value=$NATGW_NAME},{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --subnet-id $SUBNET_PUBLIC_ID \
    --allocation-id $ALLOCATION_ID \
    --region $REGION \
    --query "NatGateway.NatGatewayId" \
    --output text)
echo -e "\033[0;32mMi NAT GATEWAY::\033[0m \033[1;31m${NATGW_ID}\033[0m"
```
## 4.3 Verificar que NAT GATEWAY este disponible
```bash
aws ec2 wait nat-gateway-available \
    --nat-gateway-ids $NATGW_ID \
    --region $REGION
```
# 5 Crear Route Table privada
## 5.1 Crear Route Table
```bash
RT_PRIVATE_ID=$(aws ec2 create-route-table \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=$RT_PRIVATE_NAME},{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --vpc-id $VPC_PRIVATE_ID \
    --region $REGION \
    --query "RouteTable.RouteTableId" \
    --output text)
echo -e "\033[0;32mMi RT PRIVATE::\033[0m \033[1;31m${RT_PRIVATE_ID}\033[0m"
```
## 5.2 Crear ruta dirigida a Nat Gateway
```bash
aws ec2 create-route \
    --route-table-id $RT_PRIVATE_ID \
    --destination-cidr-bloc "0.0.0.0/0" \
    --gateway-id $NATGW_ID \
    --region $REGION
```
## 5.2 Adjuntar Subnet privada a Route Table privada
```bash
aws ec2 associate-route-table \
    --subnet-id $SUBNET_PRIVATE_ID \
    --route-table-id $RT_PRIVATE_ID \
    --region $REGION
```
# 6 Verificar
Vamos a esperar de 3-5 minutos para ver si ya se logro enviar el fichero de la instancia al bucket.
```bash
aws s3 ls s3://$S3_NAME/ --region $REGION
```
