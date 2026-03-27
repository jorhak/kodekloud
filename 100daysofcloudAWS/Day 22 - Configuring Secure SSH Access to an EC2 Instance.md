```
The Nautilus DevOps team needs to set up a new EC2 instance that can be accessed securely from their landing host (aws-client). The instance should be of type t2.micro and named xfusion-ec2. A new SSH key with name id_rsa should be created on the aws-client host under the/root/.ssh/ folder, if it doesn't already exist. This key should then be added to the root user's authorised keys on the EC2 instance, allowing passwordless SSH access from the aws-client host.


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://080366094772.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Wed Mar 11 14:35:28 UTC 2026
End Time	Wed Mar 11 15:35:28 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
REGION=us-east-1
INSTANCE_TYPE=t2.micro
INSTANCE_NAME=datacenter-ec2
IMAGE_ID=ami-0b6c6ebed2801a5cb
KEY_NAME=datacenter-key-pair
KEY_TYPE=rsa
USERNAME=ubuntu
```

1. Crear **Key pairs**:
- Verificar si existen llaves
```
ls -la ~/.ssh
```
Si no hay ningun fichero id_rsa procedemos a crear nuestra llave.

### Crear llave
- Opcion 1
	Vamos a optar por esta opcion para cumplir con la tarea
```
ssh-keygen -t rsa -b 4096
```

```
aws ec2 import-key-pair \
    --key-name $KEY_NAME \
    --public-key-material fileb://"/root/.ssh/id_rsa.pub"
```

- Opcion 2
```
aws ec2 create-key-pair \
    --key-name $KEY_NAME \
    --key-type $KEY_TYPE \
    --key-format pem \
    --tag-specifications "ResourceType=key-pair,Tags=[{Key=Entorno,Value=DEV},{Key=Llave,Value=Desarrollo}]" \
    --region $REGION \
    --query "KeyMaterial" \
    --output text > key_pair.pem
```

- Cambiar permisos
```
chmod 400 key_pair.pem
```

2. Crear **Instance**:
```
aws ec2 run-instances \
    --image-id $IMAGE_ID \
    --instance-type $INSTANCE_TYPE \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$INSTANCE_NAME}]" \
    --key-name $KEY_NAME \
    --region $REGION
```

3. Verificar estado de **Instance**

```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].InstanceId" \
    --output text)
```

- Esperar hasta que la instancia se esta ejecutando por _filters_ **Opcion 1**:
```
aws ec2 wait instance-running \
    --filters Name=tag:Name,Values=$INSTANCE_NAME
```

- Esperar hasta que la instancia se este ejecutando por _id_ **Opcion 2**:
```
aws ec2 wait instance-running \
    --instance-ids $INSTANCE_ID
```

- Estado de la **Instance**
```
aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].{ID:InstanceId,IPPrivada:PrivateIpAddress,IPPublica:PublicIpAddress,Estado:State.Name,Nombre:Tags[0].Value}" \
    --output table
```

- Obtener IP Publica
```
PUBLIC_IP=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].PublicIpAddress" \
    --output text)
```

4. Agregar regla en el **SECURITY GROUP**
- Obtener **Security Group ID**:
```
SG_NAME=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].SecurityGroups[0].GroupName" \
    --output text)
```

- Agregar regla
```
aws ec2 authorize-security-group-ingress \
    --group-name $SG_NAME \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

5. Ingresar a **Instance**:
```
ssh $USERNAME@$PUBLIC_IP
```

- Copiar _authorized_key_ a /root/.ssh
```
sudo cp ~/.ssh/authorized_keys /root/.ssh
```

- Ingresar como _root_
```
ssh root@$PUBLIC_IP
```

6. Eliminar **Instance**:
Solo si es necesario
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].InstanceId" \
    --output text)

aws ec2 terminate-instances \
    --instance-ids $INSTANCE_ID 
```


