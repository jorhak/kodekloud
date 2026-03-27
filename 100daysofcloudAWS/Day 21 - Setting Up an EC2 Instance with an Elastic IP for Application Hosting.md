```
The Nautilus DevOps Team has received a new request from the Development Team to set up a new EC2 instance. This instance will be used to host a new application that requires a stable IP address. To ensure that the instance has a consistent public IP, an Elastic IP address needs to be associated with it. The instance will be named nautilus-ec2, and the Elastic IP will be named nautilus-eip. This setup will help the Development Team to have a reliable and consistent access point for their application.

Create an EC2 instance named nautilus-ec2 using any linux AMI like ubuntu, the Instance type must be t2.micro and associate an Elastic IP address with this instance, name it as nautilus-eip.


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://557583209481.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Tue Mar 10 12:20:09 UTC 2026
End Time	Tue Mar 10 13:20:09 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
INSTANCE_NAME=devops-ec2
INSTANCE_TYPE=t2.micro
EIP_NAME=devops-eip
REGION=us-east-1
```

1. Obtener _id_ de alguna imagen **Linux**:

- Filtrando por descripcion
```
aws ec2 describe-images \
    --filters "Name=description,Values='Canonical, Ubuntu, 24.04, amd64 noble image'" \
    --region $REGION \
    --max-items 1
```

- Filtrando por id
```
aws ec2 describe-images \
    --filters Name=image-id,Values=ami-0b6c6ebed2801a5cb \
    --region $REGION \
    --max-items 1
```

Vamos a utilizar del que ya tenemos su _ImageId_
```
IMAGE_ID=ami-0b6c6ebed2801a5cb
```

2. Listar **Elastic IP**:
```
aws ec2 describe-addresses \
    --filters "Name=tag:Name,Values=$EIP_NAME" \
    --query "Addresses[].{Nombre:Tags[?Key=='Name'].Value,ID:AllocationId,IPPublica:PublicIp}" \
    --region $REGION \
    --output table
```

3. Crear **Instance**:
```
aws ec2 run-instances \
    --image-id $IMAGE_ID \
    --instance-type $INSTANCE_TYPE \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$INSTANCE_NAME}]" \
    --region $REGION
```

- Estado de la **Instance**:
```
aws ec2 describe-instances \
    --region $REGION \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[].Instances[].{Estado:State.Name,Nombre:Tags[?Key=='Name'].Value,IPPulica:PublicIpAddress,IPPrivada:PrivateIpAddress,ID:InstanceId}" \
    --output table
```
- Capturar _InstanceId_:
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --region $REGION \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query Reservations[].Instances[].InstanceId \
    --output text)
```

4. Crear **Elastic IP**:
```
aws ec2 allocate-address \
    --domain vpc \
    --tag-specifications "ResourceType=elastic-ip,Tags=[{Key=Name,Value=$EIP_NAME}]" \
    --region $REGION
```

- Capturar _AllocationId_
```
ALLOCATION_ID=$(aws ec2 describe-addresses \
    --region $REGION \
    --filters "Name=tag:Name,Values=$EIP_NAME" \
    --query "Addresses[].AllocationId" \
    --output text)
```

5. Asociar **Static IP** a **Instance**:
```
aws ec2 associate-address \
    --allocation-id $ALLOCATION_ID \
    --instance-id $INSTANCE_ID
```
