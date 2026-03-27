```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units.

For this task, create an EC2 instance with following requirements:

1) The name of the instance must be `nautilus-ec2`.

2) You can use the `Amazon Linux` AMI to launch this instance.

3) The Instance type must be `t2.micro`.

4) Create a new RSA key pair named `nautilus-kp`.

5) Attach the default (available by default) security group.  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://526831456663.signin.aws.amazon.com/console?region=us-east-1](https://526831456663.signin.aws.amazon.com/console?region=us-east-1)|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Wed Jan 21 15:23:22 UTC 2026|
|End Time|Wed Jan 21 16:23:22 UTC 2026|

  
  

  
`Notes:`

- Create the instance in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Vamos a comenzar con la variables:
```
INSTANCE_NAME=xfusion-ec2
INSTANCE_TYPE=t2.micro
RSA_KEY_NAME=xfusion-kp
REGION=us-east-1
```

Lo primero que vamos hacer sera listar **VPC**:
```
aws ec2 describe-vpcs
```

Vamos a capturar _VpcId_:
```
VPC_ID=$(aws ec2 describe-vpcs --query "Vpcs[0].VpcId" --output text)
```

Listar **Security Group**:
```
aws ec2 describe-security-groups
```

Capturar _GroupId_ y _GroupName_:
```
SECURITY_GROUP_ID=$(aws ec2 describe-security-groups --query "SecurityGroups[0].GroupId" --output text)

SECURITY_GROUP_NAME=$(aws ec2 describe-security-groups --query "SecurityGroups[0].GroupName" --output text)
```

Ahora vamos a buscar el _ImageId_ de **Amazon Linux**
```
IMAGE_ID=$(aws ec2 describe-images --region $REGION --owners amazon --filters "Name=name,Values=al2023-ami-2023.10.20260105.0-kernel-6.1-x86_64" --query "Images[0].ImageId" --output text)
```

Crear **KEY PAIR**:
```
aws ec2 create-key-pair \
    --key-name $RSA_KEY_NAME \
    --key-type rsa \
    --region $REGION \
    --tag-specifications "ResourceType=key-pair,Tags=[{Key=name,Value='$RSA_KEY_NAME'}]" \
    --query "KeyMaterial" \
    --output text > instance.pem
```

Asignar permiso de lectura:
```
chmod 400 instance.pem
```

Eliminar **Key Pair**:
```
aws ec2 delete-key-pair \
    --key-name $RSA_KEY_NAME
```

Crear **Instance** y capturamos _InstanceId_:
```
INSTANCE_ID=$(aws ec2 run-instances \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value='$INSTANCE_NAME'}]" \
    --image-id $IMAGE_ID \
    --instance-type $INSTANCE_TYPE \
    --count 1 \
    --key-name $RSA_KEY_NAME \
    --region $REGION \
    --security-group-ids $SECURITY_GROUP_ID \
    --query "Instances[0].InstanceId" \
    --output text)
```

Prueba para conectarnos por **SSH**

Agregamos regla de entrada:
```
aws ec2 authorize-security-group-ingress \
    --group-name $SECURITY_GROUP_NAME \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

Obtener **IP publica**:
```
IP_PUBLICA=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query 'Reservations[].Instances[].PublicIpAddress' \
  --output text)
```

Conexion por **SSH**:
```
ssh -i instance.pem ec2-user@$IP_PUBLICA
```

Eliminar **Instance**:
```
INSTANCE_ID=$(aws ec2 describe-instances --query "Reservations[3].Instances[0].InstanceId" --output text)

aws ec2 terminate-instances \
    --instance-ids $INSTANCE_ID 
```
