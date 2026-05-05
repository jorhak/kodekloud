```
The Nautilus DevOps Team is working on setting up a new web server for a critical application. The team lead has requested you to create an EC2 instance that will serve as a web server using Nginx. This instance will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.

As a member of the Nautilus DevOps Team, your task is to create an EC2 instance with the following specifications:

Instance Name: The EC2 instance must be named devops-ec2.

AMI: Use any available Ubuntu AMI to create this instance.

User Data Script: Configure the instance to run a user data script during its launch. This script should:

Install the Nginx package.
Start the Nginx service.
Security Group: Ensure that the instance allows HTTP traffic on port 80 from the internet.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://278810891986.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Tue May 05 13:04:12 UTC 2026
End Time	Tue May 05 14:04:12 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

### Variables de entorno
```
INSTANCE_NAME=devops-ec2
IMAGE_ID=ami-0b6c6ebed2801a5cb
INSTANCE_TYPE=t2.micro
REGION=us-east-1
KEY_NAME=millave
```

# 1 SSH
### Crear llave
```
aws ec2 create-key-pair \
    --key-name $KEY_NAME \
    --key-type rsa \
    --region $REGION \
    --tag-specifications "ResourceType=key-pair,Tags=[{Key=name,Value='$KEY_NAME'}]" \
    --query "KeyMaterial" \
    --output text > instance.pem
```

### Agregar permiso de lectura
```
chmod 400 instance.pem
```
# 2 Crear instancia EC2
### USER DATA
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
### Instancia EC2
```
INSTANCE_ID=$(aws ec2 run-instances \
    --image-id $IMAGE_ID \
    --instance-type $INSTANCE_TYPE \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$INSTANCE_NAME}]" \
    --region $REGION \
    --user-data file://user_data.yaml \
    --key-name $KEY_NAME \
    --query "Instances[0].InstanceId" \
    --output text)    
```

```
aws ec2 wait instance-running \
--instance-ids $INSTANCE_ID
```
# 3 Exponer puertos
### Capturar el nombre de security group
```
SECURITY_GROUP_NAME=$(aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query "Reservations[].Instances[].NetworkInterfaces[].Groups[].GroupName" \
--output text)
```
### Habilitar puertos 22 y 80
```
aws ec2 authorize-security-group-ingress \
    --group-name $SECURITY_GROUP_NAME \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

```
aws ec2 authorize-security-group-ingress \
    --group-name $SECURITY_GROUP_NAME \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

# 4 Verificar web
### Obtener IP publica
```
IP_PUBLICA=$(aws ec2 describe-instances \
--instance-ids $INSTANCE_ID \
--query "Reservations[].Instances[].PublicIpAddress" \
--output text)
```

### Acceder por SSH
```
ssh -i instance.pem ubuntu@$IP_PUBLICA
```
# Eliminar instancia (OPCIONAL)
```
aws ec2 terminate-instances \
--instance-ids $INSTANCE_ID
```