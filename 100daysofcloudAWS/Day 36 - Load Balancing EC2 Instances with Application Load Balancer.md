```
The Nautilus Development Team needs to set up a new EC2 instance and configure it to run a web server. This EC2 instance should be part of an Application Load Balancer (ALB) setup to ensure high availability and better traffic management. The task involves creating an EC2 instance, setting up an ALB, configuring a target group, and ensuring the web server is accessible via the ALB DNS.

Create a security group: Create a security group named devops-sg to open port 80 for the default security group (which will be attached to the ALB). Attach devops-sg security group to the EC2 instance.

Create an EC2 instance: Create an EC2 instance named devops-ec2. Use any available Ubuntu AMI to create this instance. Configure the instance to run a user data script during its launch.

This script should:
Install the Nginx package.
Start the Nginx service.

Set up an Application Load Balancer: Set up an Application Load Balancer named devops-alb. Attach default security group to the same.

Create a target group: Create a target group named devops-tg.

Route traffic: The ALB should route traffic on port 80 to port 80 of the devops-ec2 instance.

Security group adjustments: Make appropriate changes in the default security group attached to the ALB if necessary. Eventually, the Nginx server running under devops-ec2 instance must be accessible using the ALB DNS.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL https://644306594621.signin.aws.amazon.com/console?region=us-east-1

Username kk_labs_user

Password contra

Start Time Mon Aug 03 11:27:28 UTC 2026

End Time Mon Aug 03 12:27:28 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:

toggle button
```
# Variables de entorno
```
PREFIX=devops
SG_NAME=$PREFIX-sg
PORT=80
INSTANCE_NAME=$PREFIX-ec2
IMAGE_ID=ami-052355af2a014bd2c
INSTANCE_TYPE=t2.micro
RSA_KEY_NAME=$PREFIX-kp
ALB_NAME=$PREFIX-alb
TARGET=$PREFIX-tg
REGION=us-east-1
```
# Obtener IDs VPC y SG
Tenemos una VPC por default la cual tiene un SG, estos los vamos a utilizar en ALB para exponerlo.
```
VPC_ID=$(aws ec2 describe-vpcs \
    --query Vpcs[0].VpcId \
    --output text)
```

```
SG_DEFAULT=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=default" "Name=vpc-id,Values=$VPC_ID" \
  --query "SecurityGroups[0].GroupId" \
  --output text)
```
# 1 Crear Security Group
## 1.1 Crear Security Group para la instancia
```
SG_ID=$(aws ec2 create-security-group \
    --group-name $SG_NAME \
    --description "Security group for Instancias" \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "GroupId" \
    --output text)
```
## 1.2 Crear regla HTTP
```
aws ec2 authorize-security-group-ingress \
    --group-name $SG_NAME \
    --protocol tcp \
    --port $PORT \
    --cidr 0.0.0.0/0
```
# 2 Crear instancia
## 2.1 Ver imagen
```
aws ec2 describe-images \
    --image-ids $IMAGE_ID \
    --region $REGION
```
## 2.2 Crear Key Pair
```
aws ec2 create-key-pair \
    --key-name $RSA_KEY_NAME \
    --key-type rsa \
    --region $REGION \
    --tag-specifications "ResourceType=key-pair,Tags=[{Key=name,Value='$RSA_KEY_NAME'}]" \
    --query "KeyMaterial" \
    --output text > instance.pem
```
## 2.3 Dar permisos de lectura
```
chmod 400 instance.pem
```
## 2.4 Script de instalacion
```
vi user_data.yaml
```

```
#cloud-config
package_update: true
package_upgrade: true

#install packages
packages:
- nginx

#execute commands
runcmd:
- systemctl start nginx
- systemctl enable nginx
```
## 2.5 Crear instancia
```
INSTANCE_ID=$(aws ec2 run-instances \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value='$INSTANCE_NAME'}]" \
    --image-id $IMAGE_ID \
    --instance-type $INSTANCE_TYPE \
    --key-name $RSA_KEY_NAME \
    --region $REGION \
    --security-group-ids $SG_ID \
    --user-data file://user_data.yaml \
    --query "Instances[0].InstanceId" \
    --output text)
```

```
aws ec2 wait instance-running \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" "Name=instance-state-name,Values=running"
```
# 3 Crear target Group
```
TARGET_ARN=$(aws elbv2 create-target-group \
    --name $TARGET \
    --protocol HTTP \
    --port $PORT \
    --vpc-id $VPC_ID \
    --target-type instance \
    --region $REGION \
    --query "TargetGroups[0].TargetGroupArn" \
    --output text)
```
## 3.1 Registrar la instancia
```
aws elbv2 register-targets \
    --target-group-arn $TARGET_ARN \
    --targets Id=$INSTANCE_ID
```
# 4 Crear Load Balancer
## 4.1 Configurar Security Group (default) para ALB
```
aws ec2 authorize-security-group-ingress \
  --group-id $SG_DEFAULT \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```
## 4.2 Obtener Subnet's de VPC
```
aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query "Subnets[*].[SubnetId,AvailabilityZone]" \
  --output table
```
## 4.3 Crear Load Balancer
Debemos actualizar los parametros de **--subnets** porque estos pueden cambiar.
```
ALB_ARN=$(aws elbv2 create-load-balancer \
  --name $ALB_NAME \
  --type application \
  --security-groups $SG_ID \
  --subnets subnet-0ba56ded21739b4cf subnet-079958baf28eb74f4 subnet-07aa30e74984b187f subnet-04f1788d4c05d24d3 subnet-085d34d5cacddb6cd subnet-033bf6d30cdb6b28a \
  --region $REGION \
  --query "LoadBalancers[0].LoadBalancerArn" \
  --output text)
```
## 4.4 Crear Listener
```
aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=$TARGET_ARN
```
# 5 Verificar
Debemos esperar unos 5 minutos para la propagacion de DNS
```
aws elbv2 describe-load-balancers \
  --names $LB_NAME \
  --query "LoadBalancers[0].DNSName" \
  --output text
```
Se debe ejecutar despues de la creacion de la instancia paso **2.5**
```
aws ec2 describe-instance-attribute \
  --instance-id $INSTANCE_ID \
  --attribute userData \
  --region us-east-1 \
  --query "UserData.Value" \
  --output text
```