```
The DevOps team is tasked with setting up a highly available web application using AWS. To achieve this, they plan to use an Auto Scaling Group (ASG) to ensure that the required number of EC2 instances are always running, and an Application Load Balancer (ALB) to distribute traffic across these instances. The goal of this task is to set up an ASG that automatically scales EC2 instances based on CPU utilization, and an ALB that directs incoming traffic to the instances. The EC2 instances should have Nginx installed and running to serve web traffic.

Create an EC2 launch template named datacenter-launch-template that specifies the configuration for the EC2 instances, including the Amazon Linux 2023 AMI, t2.micro instance type, and a security group that allows HTTP traffic on port 80.
Add a User Data script to the launch template to install Nginx on the EC2 instances when they are launched. The script should install Nginx, start the Nginx service, and enable it to start on boot.
Create an Auto Scaling Group named datacenter-asg that uses the launch template and ensures a minimum of 1 instance, desired capacity is 1 instance and a maximum of 2 instances are running based on CPU utilization. Set the target CPU utilization to 50%.
Create a target group named datacenter-tg, an Application Load Balancer named datacenter-alb and configure it to listen on port 80. Ensure the ALB is associated with the Auto Scaling Group and distributes traffic across the instances.
Configure health checks on the ALB to ensure it routes traffic only to healthy instances.
Verify that the ALB's DNS name is accessible and that it displays the default Nginx page served by the EC2 instances.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://860767964813.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Fri Aug 21 02:45:44 UTC 2026
End Time	Fri Aug 21 03:45:44 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
PREFIX=devops
LAUNCH_TEMPLATE_NAME=$PREFIX-launch-template
INSTANCE_TYPE=t2.micro
SG_NAME=$PREFIX-sg
SG_PORT_HTTP=80
SG_PORT_SSH=22
ASG_NAME=$PREFIX-asg
MIN_INSTANCE=1
MAX_INSTANCE=2
CPU_UTILIZATION=50
TG_NAME=$PREFIX-tg
ALB_NAME=$PREFIX-alb
ALB_PORT=80
REGION=us-east-1
KEY_NAME=$PREFIX-key-pair
ASG_POLICY_NAME=$PREFIX-asg-policy
```
# Buscar el _ImageId_ de **Amazon Linux**
```
IMAGE_ID=$(aws ec2 describe-images \
    --filters Name=name,Values=al2023-ami-2023.12.20260727.0-kernel-6.1-x86_64 \
    --owners amazon \
    --region $REGION \
    --query "Images[].ImageId" \
    --output text)
echo -e "\033[0;32mMi Image ID::\033[0m \033[1;31m${IMAGE_ID}\033[0m"
```
# 1 Crear VPC
```
VPC_ID=$(aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications "ResourceType=vpc,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --query "Vpc.VpcId" \
    --output text \
    --region $REGION)
echo -e "\033[0;32mMi VPC::\033[0m \033[1;31m${VPC_ID}\033[0m"
```
### Opcional
```
VPC_ID=$(aws ec2 describe-vpcs \
    --filters Name=tag:Env,Values=DEV Name=tag:IT,Values=SALESCARE \
    --query Vpcs[0].VpcId \
    --output text)
echo -e "\033[0;32mMi VPC Opcional::\033[0m \033[1;31m${VPC_ID}\033[0m"
```
# 2 Crear Subnets
```
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --availability-zone ${REGION}a \
    --cidr-block 10.0.1.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]"
```

```
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --availability-zone ${REGION}b \
    --cidr-block 10.0.2.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]"
```
##  2.1 Obtener Subnet's de la VPC
```
SUBNET_IDS=$(aws ec2 describe-subnets \
    --filters Name=vpc-id,Values=$VPC_ID \
    --query "Subnets[*].SubnetId" \
    --output text \
    --region $REGION)
```

```
SUBNET_1=$(echo $SUBNET_IDS | awk '{print $1}') 
SUBNET_2=$(echo $SUBNET_IDS | awk '{print $2}') 
SUBNET_LIST_COMMAS=$(echo $SUBNET_IDS | tr ' ' ',')
```
## 2.2 Habilitar IP Publica
```
aws ec2 modify-subnet-attribute \
    --map-public-ip-on-launch \
    --subnet-id $SUBNET_1 \
    --region $REGION
    
aws ec2 modify-subnet-attribute \
    --map-public-ip-on-launch \
    --subnet-id $SUBNET_2 \
    --region $REGION
```
# 3 Crear Internet Gateway
```
IGW_ID=$(aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --region $REGION \
    --query 'InternetGateway.InternetGatewayId' \
    --output text)
echo -e "\033[0;32mMi IGW::\033[0m \033[1;31m${IGW_ID}\033[0m"
```
## 3.1 Adjuntar Internet Gateway a VPC
```
aws ec2 attach-internet-gateway \
    --vpc-id $VPC_ID \
    --internet-gateway-id $IGW_ID \
    --region $REGION
```
# 4 Enrutamiento
## 4.1 Crear tabla de rutas
```
RT_ID=$(aws ec2 create-route-table \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query "RouteTable.RouteTableId" \
    --output text)
```
## 4.2 Crear Route hacia el IGW
```
aws ec2 create-route \
    --route-table-id $RT_ID \
    --destination-cidr-bloc 0.0.0.0/0 \
    --gateway-id $IGW_ID \
    --region $REGION
```
## 4.3 Adjuntar Route Table a nuestras Subnet's
```
aws ec2 associate-route-table \
    --subnet-id $SUBNET_1 \
    --route-table-id $RT_ID \
    --region $REGION
    
aws ec2 associate-route-table \
    --subnet-id $SUBNET_2 \
    --route-table-id $RT_ID \
    --region $REGION
```
# 5 Crear Security Group
```
SG_ID=$(aws ec2 create-security-group \
    --description "Security Group de mis instancias High Avility" \
    --group-name $SG_NAME \
    --vpc-id $VPC_ID \
    --tag-specifications "ResourceType=security-group,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --region $REGION \
    --query "GroupId" \
    --output text)
echo -e "\033[0;32mMi Security Group::\033[0m \033[1;31m${SG_ID}\033[0m"
```
### OPCIONAL
```
SG_ID=$(aws ec2 describe-security-groups \
    --filters Name=tag:Env,Values=DEV Name=tag:IT,Values=SALESCARE \
    --query "SecurityGroups[0].GroupId" \
    --output text)
echo -e "\033[0;32mMi Security Group Opcional::\033[0m \033[1;31m${SG_ID}\033[0m"
```
## 5.1 Crear regla HTTP
```
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port $SG_PORT_HTTP \
    --tag-specifications "ResourceType=security-group-rule,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --cidr 0.0.0.0/0 \
    --region $REGION
```
## 5.2 Crear regla SSH
```
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port $SG_PORT_SSH \
    --tag-specifications "ResourceType=security-group-rule,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --cidr 0.0.0.0/0 \
    --region $REGION
```
# 6 Crear Key Pair
```
aws ec2 create-key-pair \
    --key-name $KEY_NAME \
    --key-type "rsa" \
    --query "KeyMaterial" \
    --tag-specifications "ResourceType=key-pair,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --output text > template_instances.pem
```

## 6.1 Agregar permiso de lectura
```
chmod 400 template_instances.pem
```
# 7 Crear Launch Template
## 7.1 Crear User Data Script
```
cat<<EOF >user_data.yaml 
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
EOF
USER_DATA=$(cat user_data.yaml | base64 | tr -d '\n')
echo -e "\033[0;32mMi user data::\033[0m \033[1;31m${USER_DATA}\033[0m"
```

## 7.2 Crear fichero de configuracion

```
cat<<EOF > template_data.json
{
    "ImageId": "$IMAGE_ID",
    "InstanceType": "$INSTANCE_TYPE",
    "KeyName": "$KEY_NAME",
    "SecurityGroupIds": ["$SG_ID"],
    "UserData":"$USER_DATA",
    "TagSpecifications": [
        {
            "ResourceType": "instance",
            "Tags": [
                {
                    "Key": "Env",
                    "Value": "DEV"
                },
                {
                    "Key": "IT",
                    "Value": "SALESCARE"
                }
            ]
        }
    ]
}
EOF
```
# 7.3 Crear Launch Template
```
aws ec2 create-launch-template \
    --launch-template-name $LAUNCH_TEMPLATE_NAME \
    --launch-template-data file://template_data.json \
    --version-description "v1 - Auto escalado de instancias" \
    --tag-specifications "ResourceType=launch-template,Tags=[{Key=Env,Value=DEV},{Key=IT,Value=SALESCARE}]" \
    --region $REGION
```
### Opcional
```
aws ec2 delete-launch-template \
    --launch-template-name $LAUNCH_TEMPLATE_NAME
```
# 8 Crear Target Group
```
TG_ARN=$(aws elbv2 create-target-group \
    --name $TG_NAME \
    --protocol HTTP \
    --port 80 \
    --vpc-id $VPC_ID \
    --target-type instance \
    --health-check-protocol HTTP \
    --health-check-port 80 \
    --health-check-path "/" \
    --query "TargetGroups[0].TargetGroupArn" \
    --output text \
    --region $REGION)
```
# 9 Crear Application Load Balancer
```
ALB_ARN=$(aws elbv2 create-load-balancer \
    --name $ALB_NAME \
    --subnets $SUBNET_1 $SUBNET_2 \
    --security-groups $SG_ID \
    --scheme internet-facing \
    --type application \
    --query "LoadBalancers[0].LoadBalancerArn" \
    --output text \
    --region $REGION)
```
# 10 Crear Listener
```
aws elbv2 create-listener \
    --load-balancer-arn $ALB_ARN \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=$TG_ARN \
    --region $REGION
```
# 11 Crear Auto Scaling Group
```
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name $ASG_NAME \
    --launch-template LaunchTemplateName=$LAUNCH_TEMPLATE_NAME \
    --min-size $MIN_INSTANCE \
    --max-size $MAX_INSTANCE \
    --desired-capacity $MIN_INSTANCE \
    --vpc-zone-identifier "$SUBNET_LIST_COMMAS" \
    --target-group-arns $TG_ARN \
    --health-check-type ELB \
    --health-check-grace-period 300 \
    --region $REGION
```
## 11.1 Crear politica de escalado
```
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name $ASG_NAME \
    --policy-name $ASG_POLICY_NAME \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
      "PredefinedMetricSpecification": {
        "PredefinedMetricType": "ASGAverageCPUUtilization"
      },
      "TargetValue": 50.0
    }' \
    --region $REGION
```
# Verificacion
## Obtenemos el DNS del Application Load Balancer
```
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --names $ALB_NAME \
  --query "LoadBalancers[0].DNSName" \
  --output text --region $REGION)

echo -e "\033[0;32mDNS del ALB:\033[0m \033[1;31m http://$ALB_DNS\033[0m"
```
## Obtenemos las IP publicas de las instancias
```
aws ec2 describe-instances \
    --filters Name=tag:Env,Values=DEV Name=tag:IT,Values=SALESCARE Name=instance-state-name,Values=running \
    --query "Reservations[].Instances[*].{IPPublica:PublicIpAddress,IPPrivate:PrivateIpAddress}" \
    --output table
```
### Ingresar a los servidores para ver las peticiones
Puede que solo se haya creado una instancias por la politica por eso vamos a realizar peticiones y vamos a estar atentos para conectarnos a esa instancia y ver si le llegan las peticiones
```
ssh -i template_instances.pem ec2-user@<IP Publica del servidor>
```

```
sudo tail -f /var/log/nginx/access.log
```
## Estresar CPU para que se cree la instancia
```
sudo yum install stress-ng -y
stress-ng --cpu 0 --cpu-load 60 --timeout 5m
```
## Probar con CURL
```
for i in {1..100}
do
   curl -I http://$ALB_DNS
done
```
