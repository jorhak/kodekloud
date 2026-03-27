```
The Nautilus DevOps team is currently working on setting up a simple application on the AWS cloud. They aim to establish an Application Load Balancer (ALB) in front of an EC2 instance where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later.

1. Set up an Application Load Balancer named `xfusion-alb`.
2. Create a target group named `xfusion-tg`.
3. Create a security group named `xfusion-sg` to open port `80` for the public.
4. Attach this security group to the ALB.
5. The ALB should route traffic on port `80` to port `80` of the `xfusion-ec2` instance.
6. Make appropriate changes in the default security group attached to the EC2 instance if necessary.  
      
    

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://453274592256.signin.aws.amazon.com/console?region=us-east-1]|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Thu Mar 26 13:59:09 UTC 2026|
|End Time|Thu Mar 26 14:59:09 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Variables de entorno:
```
ALB_NAME=nautilus-alb
ALB_TG_NAME=nautilus-tg
ALB_SG_NAME=nautilus-sg
INSTANCE_NAME=nautilus-ec2
REGION=us-east-1
```

1. Obtener _id_ de **VPC**
```
VPC_ID=$(aws ec2 describe-vpcs --query "Vpcs[0].VpcId" --output text)
```

2. Crear _security group_ de **ALB**
```
ALB_SG_ID=$(aws ec2 create-security-group \
    --description "Security Group para ALB"\
    --group-name $ALB_SG_NAME \
    --vpc-id $VPC_ID \
    --region $REGION \
    --query GroupId \
    --output text)
```

```
aws ec2 wait security-group-exists --group-ids "$ALB_SG_ID"
```

- Una ves creado el **Security Group** debemos darle las reglas de entrada:
```
aws ec2 authorize-security-group-ingress \
    --group-id $ALB_SG_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

- Obtener _id_ de **Security Group** de la instancia
```
INSTANCE_SG_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[].Instances[].SecurityGroups[].GroupId" \
    --output text)
```

3. Permitir trafico entre **ALB** y la **INSTANCE**
```
aws ec2 authorize-security-group-ingress \
    --group-id $INSTANCE_SG_ID \
    --protocol tcp \
    --port 80 \
    --source-group $ALB_SG_ID
```

4. Crear _target group_
```
TG_ARN=$(aws elbv2 create-target-group \
    --name $ALB_TG_NAME \
    --protocol HTTP \
    --port 80 \
    --vpc-id $VPC_ID \
    --target-type instance \
    --region $REGION \
    --query TargetGroups[].TargetGroupArn \
    --output text)
```

- Obtener _id_ de _instance_
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query Reservations[0].Instances[0].InstanceId \
    --output text)
```

- Registrar _instancia_ en el _target group_
```
aws elbv2 register-targets \
    --target-group-arn $TG_ARN \
    --targets Id=$INSTANCE_ID,Port=80 \
    --region $REGION
```

4. Crear _load balancer_
	Se deben agregar al menos dos subnets en zonas de disponibilidad diferentes
	**SUBNET_ID_1** y **SUBNET_ID_2**.
	
- Listar las subnets
```
aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$VPC_ID" \
    --query "Subnets[*].{ID:SubnetId,AZ:AvailabilityZone,CIDR:CidrBlock}" \
    --output table
```

- Obtener una subnet
```
SUBNETS_IDS=$(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$VPC_ID" \
    --query "Subnets[0:1].SubnetId" \
    --output text)
```

- Obtener **AZ** y **SUBNET** de la _instancia_
	Debemos agregar la subnet donde se encuentra la instancia
```
INSTANCE_AZ=$(aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query "Reservations[0].Instances[0].Placement.AvailabilityZone" \
--output text)
```

```
NUEVA_SUBNET_ID=$(aws ec2 describe-subnets \
--filters "Name=availability-zone,Values=$INSTANCE_AZ" "Name=vpc-id,Values=$VPC_ID" \
--query "Subnets[0].SubnetId" \
--output text)
```

- Crear load balancer
```
ALB_ARN=$(aws elbv2 create-load-balancer \
    --name $ALB_NAME \
    --subnets $SUBNETS_IDS $NUEVA_SUBNET_ID\
    --security-groups $ALB_SG_ID \
    --region $REGION \
    --type application \
    --scheme internet-facing \
    --query LoadBalancers[].LoadBalancerArn \
    --output text)
```

```
aws elbv2 wait load-balancer-exists \
    --load-balancer-arn $ALB_ARN
```

5. Crear listener para redirigir el trafico del puerto 80 al Target Group
```
aws elbv2 create-listener \
    --load-balancer-arn $ALB_ARN \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=$TG_ARN
```

6. Verificar
```
aws elbv2 describe-load-balancers \
    --names $ALB_NAME \
    --query 'LoadBalancers[0].DNSName' --output text
```

```
aws elbv2 describe-load-balancers \
    --load-balancer-arns $ALB_ARN \
    --query "LoadBalancers[0].{DNS:DNSName,Nombre:LoadBalancerName,Estado:State.Code}" \
    --output table
```
En el estado debe pasar de _provisioning_ a _active_
Hay que esperar unos 2 a 5 minutos.

7. Eliminar load balancer (OPCIONAL)
```
aws elbv2 delete-load-balancer \
    --load-balancer-arn $ALB_ARN
```

8. Estado de la _instance_
```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[].Instances[].{IPPublica:PublicIpAddress,IPPrivada:PrivateIpAddress,Estado:State.Name,Nombre:Tags[].Value,SG:SecurityGroups[].GroupId,NIC:NetworkInterfaces[].NetworkInterfaceId}" \
    --output table
```
