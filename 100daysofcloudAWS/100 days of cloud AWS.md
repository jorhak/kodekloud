Vamos a realizar el reto de 100 dias de nube.

# Day 1: Create Key Pair
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a key pair with the following requirements:

- Name of the `key pair` should be `datacenter-kp`.
  
- Key pair `type` must be `rsa`
  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://503702267784.signin.aws.amazon.com/console?region=us-east-1](https://503702267784.signin.aws.amazon.com/console?region=us-east-1)|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Fri Jan 16 14:06:24 UTC 2026|
|End Time|Fri Jan 16 15:06:24 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Lo primero que vamos hacer es ver que llaves estan creadas:
```
aws ec2 describe-key-pairs
```

Ahora vamos a crear las llaves:
```
aws ec2 create-key-pair \
    --key-name "datacenter-kp" \
    --key-type "rsa" \
    --query "KeyMaterial" \
    --tag-specifications 'ResourceType=key-pair,Tags=[{Key=Desarrollo,Value=Dev},{Key=PreProduccion,Value=Staging}]' \
    --output text > mi-primer-llave.pem
```

Vamos a borrar las llaves
```
aws ec2 delete-key-pair --key-name "datacenter-kp-1"
```

# Day 2: Create Security Group

```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a security group under default VPC with the following requirements:

Name of the security group is datacenter-sg.

The description must be Security group for Nautilus App Servers

Add the inbound rule of type HTTP, with port range of 80. Enter the source CIDR range of 0.0.0.0/0.

Add another inbound rule of type SSH, with port range of 22. Enter the source CIDR range of 0.0.0.0/0.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://254222094606.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Sat Jan 17 13:55:28 UTC 2026
End Time	Sat Jan 17 14:55:28 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Lo primero que vamos hacer sera identificar la VPC por defecto:
```
aws ec2 describe-vpcs
```

Del resultado que nos salga debemos obtener el _VpcId_. Despues vamos a crear el **Security Group**:
```
aws ec2 create-security-group \
    --group-name "datacenter-sg" \
    --description "Security group for Nautilus App Servers" \
    --vpc-id "vpc-0f99fc0353847efce"
```

Una ves creado el **Security Group** debemos darle las reglas de entrada:
```
aws ec2 authorize-security-group-ingress \
    --group-name "datacenter-sg" \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
    
aws ec2 authorize-security-group-ingress \
    --group-name "datacenter-sg" \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

Verificar que se creo **Security Group**:
```
aws ec2 describe-security-groups
```

# Day 3: Create Subnet
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition.

For this task, create one subnet named `nautilus-subnet` under default VPC.  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://993507109166.signin.aws.amazon.com/console?region=us-east-1](https://993507109166.signin.aws.amazon.com/console?region=us-east-1)|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Mon Jan 19 17:57:33 UTC 2026|
|End Time|Mon Jan 19 18:57:33 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Lo primero que vamos hacer sera listar la VPC para saber cual esta por defecto:
```
aws ec2 describe-vpcs
```

Y vamos a obtener el VpcId:
```
VPC_ID=$(aws ec2 describe-vpcs --query "Vpcs[*].{VpcId:VpcId}" --output text)
SN_NAME=nautilus-subnet
LOCATION=us-east-1a
```

Listar las subnet para ver sus CIDR
```
aws ec2 describe-subnets
```

Lo segundo que vamos hacer sera crear la subnet:
```
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --availability-zone $LOCATION \
    --cidr-block 172.31.124.0/20 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=$SN_NAME}]"
```

# Day 4: Enable Versioning for S3 Bucket
```
Data protection and recovery are fundamental aspects of data management. It's essential to have systems in place to ensure that data can be recovered in case of accidental deletion or corruption. The DevOps team has received a requirement for implementing such measures for one of the S3 buckets they are managing.

The s3 bucket name is datacenter-s3-12881, enable versioning for this bucket.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://995698815753.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Tue Jan 20 17:27:50 UTC 2026
End Time	Tue Jan 20 18:27:50 UTC 2026

Notes:

Create the resources only in us-east-1 region.
To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:\ntoggle button
```

Vamos a comenzar creando las variables:
```
S3_NAME=datacenter-s3-12881
REGION=us-east-1
```

Crear **Bucket**:
```
aws s3api put-bucket-versioning \
    --bucket $S3_NAME \
    --region $REGION \
    --versioning-configuration Status=Enabled
```

Verificar que se creo el **Bucket**
```
aws s3api list-buckets
```

# Day 5: Create GP3 Volume
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

Create a volume with the following requirements:

- Name of the volume should be `xfusion-volume`.
  
- Volume `type` must be `gp3`.
  
- Volume `size` must be `2 GiB`.
  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://919171157955.signin.aws.amazon.com/console?region=us-east-1](https://919171157955.signin.aws.amazon.com/console?region=us-east-1)|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Tue Jan 20 18:39:05 UTC 2026|
|End Time|Tue Jan 20 19:39:05 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Vamos a comenzar con la variables:
```
VOLUME_NAME=xfusion-volume
REGION=us-east-1
AZ=us-east-1a
```

Crear volumen:
```
aws ec2 create-volume \
    --availability-zone $AZ \
    --volume-type gp3 \
    --size 2 \
    --region $REGION \
    --tag-specifications "ResourceType=volume,Tags=[{Key=Name,Value='$VOLUME_NAME'}]"
```

Listar volumen creado:
```
aws ec2 describe-volumes
```

# Day 6: Launch EC2 Instance
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

# Day 7: Change EC2 Instance Type
```
During the migration process, the Nautilus DevOps team created several EC2 instances in different regions. They are currently in the process of identifying the correct resources and utilization and are making continuous changes to ensure optimal resource utilization. Recently, they discovered that one of the EC2 instances was underutilized, prompting them to decide to change the instance type. Please make sure the `Status check` is completed (if its still in `Initializing` state) before making any changes to the instance.

1) Change the instance type from `t2.micro` to `t2.nano` for `datacenter-ec2` instance.

2) Make sure the ec2 instance `datacenter-ec2` is in `running` state after the change.  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://113749894636.signin.aws.amazon.com/console?region=us-east-1](https://113749894636.signin.aws.amazon.com/console?region=us-east-1)|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Thu Jan 22 13:52:49 UTC 2026|
|End Time|Thu Jan 22 14:52:49 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Lo primero que vamos hacer sera ver las **Instancias**:
```
aws ec2 describe-instances
```

Vamos a capturar **InstanceId**:
```
INSTANCE_ID=$(aws ec2 describe-instances --query "Reservations[0].Instances[0].InstanceId" --output text)
```

Vamos a detener la instancia:
```
aws ec2 stop-instances \
    --instance-ids $INSTANCE_ID
```

Modificar el tipo de instancia:
```
TYPE=t2.nano
aws ec2 modify-instance-attribute \
    --instance-id $INSTANCE_ID \
    --instance-type "{\"Value\": \"$TYPE\"}"
```

Reiniciamos instancia:
```
aws ec2 start-instances \
    --instance-ids $INSTANCE_ID
```

Ver estado:
```
aws ec2 describe-instances --query "Reservations[0].Instances[0].State"
#### o
aws ec2 describe-instance-status \
    --instance-ids $INSTANCE_ID
```

# Day 8: Enable Stop Protection for EC2 Instance
```
As part of the migration, there were some components added to the AWS account. Team created one of the EC2 instances where they need to make some changes now.

There is an EC2 instance named devops-ec2 under us-east-1 region, enable the stop protection for this instance.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://111727141030.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Thu Jan 22 18:49:41 UTC 2026
End Time	Thu Jan 22 19:49:41 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Lo primero que vamos hacer sera listar las **Instances**:
```
aws ec2 describe-instances
```

Capturamos su _InstanceId_:
```
INSTANCE_ID=$(aws ec2 describe-instances --query "Reservations[].Instances[].InstanceId" --output text)
```

Vamos a habilitar **Stop detection**:
```
aws ec2 modify-instance-attribute \
    --instance-id $INSTANCE_ID \
    --disable-api-stop
```

Verificar los cambios:
```
aws ec2 describe-instance-attribute --instance-id $INSTANCE_ID --attribute disableApiStop
```

# Day 9: Enable Termination Protection for EC2 Instance
```
As part of the migration, there were some components created under the AWS account. The Nautilus DevOps team created one EC2 instance where they forgot to enable the termination protection which is needed for this instance.

An instance named devops-ec2 already exists in us-east-1 region. Enable termination protection for the same.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://316890205783.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Fri Feb 06 20:07:07 UTC 2026
End Time	Fri Feb 06 21:07:07 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
I_NAME=nautilus-ec2
REGION=us-east-1
```

Listrar **Instance**:
```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$I_NAME" \
    --query "Reservations[*].Instances[*].{ID:InstanceId,Name:Tags[?Key=='Name'].Value|[0],State:State.Name,IP:PrivateIpAddress}" \
    --output table
```

Capturarmos _InstanceId_:
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$I_NAME" \
    --query "Reservations[*].Instances[*].{ID:InstanceId}" \
    --output text)
```

Ver el estado actual de **Termination protection**:
```
aws ec2 describe-instance-attribute \
    --instance-id $INSTANCE_ID \
    --attribute disableApiTermination
```

Habilitar **Termination protection**:
```
aws ec2 modify-instance-attribute \
    --instance-id $INSTANCE_ID \
    --disable-api-termination "{\"Value\": true}"
```

Para verificar volvemos a ejecutar: Ver el estado actual de **Termi....**.

# Day 10: Attach Elastic IP to EC2 Instance
```
The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

There is an instance named devops-ec2 and an elastic-ip named devops-ec2-eip in us-east-1 region. Attach the devops-ec2-eip elastic-ip to the devops-ec2 instance.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://915322164324.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Thu Feb 19 15:13:26 UTC 2026
End Time	Thu Feb 19 16:13:26 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
INSTANCE_NAME=xfusion-ec2
EIP_NAME=xfusion-ec2-eip
REGION=us-east-1
```

Listar **Instances**:
```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[*].Instances[*].{ID:InstanceId,Name:Tags[?Key=='Name'].Value|[0],State:State.Name,IP:PrivateIpAddress}" \
    --output table
```

Obtener **InstanceId**:
```
INSTANCE_ID=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=$INSTANCE_NAME" \
--query "Reservations[].Instances[].InstanceId" \
--output text --region $REGION)
```

Capturar _AllocationId_:
```
ALLOCATION_ID=$(aws ec2 describe-addresses \
--filters "Name=tag:Name,Values=$EIP_NAME" \
--query Addresses[].AllocationId \
--output text --region $REGION)
```

Adjuntar **Elastic IP**:
```
aws ec2 associate-address \
    --instance-id $INSTANCE_ID \
    --allocation-id $ALLOCATION_ID \
    --region $REGION
```

# Day 11: Attach Elastic Network Interface to EC2 Instance
```
The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

An instance named nautilus-ec2 and an elastic network interface named nautilus-eni already exists in us-east-1 region.

Attach the nautilus-eni network interface to the nautilus-ec2 instance.
Make sure status is attached before submitting the task.
Please make sure instance initialisation has been completed before submitting this task.



Use below given AWS Credentials. (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://037671897032.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Wed Feb 25 13:47:51 UTC 2026
End Time	Wed Feb 25 14:47:51 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

1. Variables de entorno
```
INSTANCE_NAME=nautilus-ec2
ENI_NAME=nautilus-eni
REGION=us-east-1
```

2. Listar **Instance**:
```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[*].Instances[*].{ID:InstanceId,Name:Tags[?Key=='Name'].Value|[0],State:State.Name,NetworkInterfaces[*]IPPrivada:PrivateIpAddress,IPPublica:PublicIpAddress}" \
    --output table
```

3. Listar las **Network Interface** de la instancia:
```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[*].Instances[*].NetworkInterfaces[*].{IPPrivada:PrivateIpAddress}" \
    --output table
```

3. Obtener _InstanceId_:
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[*].Instances[*].{ID:InstanceId}" \
    --output text)
```

4. Listar **Elastic Network Interface**:
```
aws ec2 describe-network-interfaces \
    --filters "Name=tag:Name,Values=$ENI_NAME" \
    --query "NetworkInterfaces[*].{Descripcion:Description,IPPrivada:PrivateIpAddress}" \
    --output table
```

5. Obtener _NetworkInterfaceId_:
```
NI_ID=$(aws ec2 describe-network-interfaces \
    --filters "Name=tag:Name,Values=$ENI_NAME" \
    --query "NetworkInterfaces[*].{RedID:NetworkInterfaceId}" \
    --output text)
```

6. Conectar **Network Interface**:
```
aws ec2 attach-network-interface \
    --network-interface-id $NI_ID \
    --instance-id $INSTANCE_ID \
    --device-index 1
```

# Day 12: Attach Volume to EC2 Instance
```
The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

An instance named devops-ec2 and a volume named devops-volume already exists in us-east-1 region. Attach the devops-volume volume to the devops-ec2 instance, make sure to set the device name to /dev/sdb while attaching the volume.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://644306594621.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user_264113
Password	ygL64s!rdaNp
Start Time	Thu Feb 26 17:48:58 UTC 2026
End Time	Thu Feb 26 18:48:58 UTC 2026
Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
INSTANCE_NAME=devops-ec2
VOLUME_NAME=devops-volume
REGION=us-east-1
DEVICE_NAME=/dev/sdb
```

1. Ver instancia:
```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[].Instances[].{NOMBRE:Tags[?Key=='Name'].Value,ID_INSTANCIA:InstanceId}" \
    --output table
```

1. Ver volumen:
```
aws ec2 describe-volumes \
    --filters "Name=tag:Name,Values=$VOLUME_NAME" \
    --query "Volumes[].{NOMBRE:Tags[?Key=='Name'].Value,VOLUME_ID:VolumeId}" \
    --output table
```

2. Capturar _InstanceId_
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[].Instances[].InstanceId" \
    --output text)
```

3. Capturar _VolumeId_
```
VOLUME_ID=$(aws ec2 describe-volumes \
--filters "Name=tag:Name,Values=$VOLUME_NAME" \
--query "Volumes[].VolumeId" \
--output text) 
```

4. Adjuntar volumen
```
aws ec2 attach-volume \
    --device $DEVICE_NAME \
    --instance-id $INSTANCE_ID \
    --volume-id $VOLUME_ID
```

# Day 13: Create AMI from EC2 Instance
```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create an AMI from an existing EC2 instance named datacenter-ec2 with the following requirement:

Name of the AMI should be datacenter-ec2-ami, make sure AMI is in available state.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://093501631776.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Fri Feb 27 13:03:42 UTC 2026
End Time	Fri Feb 27 14:03:42 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
INSTANCE_NAME=xfusion-ec2
AMI_NAME=xfusion-ec2-ami
REGION=us-east-1
```

1. Ver instancia"
```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[].Instances[].{NOMBRE:Tags[?Key=='Name'].Value, ID_INSTANCIA:InstanceId}" \
    --output table
```

2. Obtener _InstanceId_:
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query Reservations[].Instances[].InstanceId \
    --output text)
```

3. Crear **AMI**:
```
aws ec2 create-image \
    --instance-id $INSTANCE_ID \
    --name $AMI_NAME \
    --description "Creando imagen de una instancia EC2" \
    --region $REGION \
    --no-reboot
```
Capturarmos el _ImageId_ en una variable de entorno 
```
IMAGE_ID=ami-xxxxxxxxxxxxxxxx
```

4. Verificar que se creo **AMI**:
```
aws ec2 describe-images \
    --image-id $IMAGE_ID
```

5. Ver estado:
```
aws ec2 describe-images \
    --image-id $IMAGE_ID \
    --filters "Name=name,Values=$AMI_NAME" \
    --query "Images[].{Descripcion:Description,Nombre:Name,ID:ImageId,Estado:State}" \
    --output table
```


https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html

https://docs.aws.amazon.com/es_es/toolkit-for-visual-studio/latest/user-guide/tkv-create-ami-from-instance.html

https://docs.aws.amazon.com/cli/latest/reference/ec2/create-image.html

# Day 14: Terminate EC2 Instance
```
During the migration process, several resources were created under the AWS account. Later on, some of these resources became obsolete as alternative solutions were implemented. Similarly, there is an instance that needs to be deleted as it is no longer in use.

1) Delete the ec2 instance named xfusion-ec2 present in us-east-1 region.

2) Before submitting your task, make sure instance is in terminated state.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://081866666028.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Sat Feb 28 13:33:43 UTC 2026
End Time	Sat Feb 28 14:33:43 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
INSTANCE_NAME=xfusion-ec2
REGION=us-east-1
```

1. Ver instancia:
```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[].Instances[].{NOMBRE:Tags[?Key=='Name'].Value, ID_INSTANCIA:InstanceId,ESTADO:State.Name}" \
    --output table
```

2. Obtener _InstaceId_:
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[].Instances[].InstanceId" \
    --output text)
```

3. Eliminar instancia:
```
aws ec2 terminate-instances \
    --instance-id $INSTANCE_ID \
    --region $REGION
```

# Day 15: Create Volume Snapshot 
```
The Nautilus DevOps team has some volumes in different regions in their AWS account. They are going to setup some automated backups so that all important data can be backed up on regular basis. For now they shared some requirements to take a snapshot of one of the volumes they have.
Create a snapshot of an existing volume named datacenter-vol in us-east-1 region.
1) The name of the snapshot must be datacenter-vol-ss.
2) The description must be datacenter Snapshot.
3) Make sure the snapshot status is completed before submitting the task.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)
Console URL
https://143588534152.signin.aws.amazon.com/console?region=us-east-1
Username
kk_labs_user
Password


Start Time
Sun Mar 01 11:45:13 UTC 2026
End Time
Sun Mar 01 12:45:13 UTC 2026


Notes:
Create the resources only in us-east-1 region.
To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:


```

Variables de entorno
```
VOLUME_NAME=datacenter-vol
SNAPSHOT_NAME=datacenter-vol-ss
SNAPSHOT_DESCRIPTION=”datacenter Snapshot”
REGION=us-east-1
``` 

1. Capturar _volumeId_:
```
VOLUME_ID=$(aws ec2 describe-volumes \
       --filters “Name=tag:Name,Values=$VOLUME_NAME” \
       --query “Volumes[].VolumeId” \
       --output text)
```

2. Crear **Snapshot**:
```
aws ec2 create-snapshot \
--volume-id $VOLUME_ID \
--description “$SNAPSHOT_DESCRIPTION” \
--tag-specifications “ResourceType=snapshot,Tags=[{Key=Name,Value=$SNAPSHOT_NAME}]” \
--region $REGION
```

3. Estado de **Snapshot**:
```
aws ec2 describe-snapshots --filters "Name=tag:Name,Values=$SNAPSHOT_NAME" --query "Snapshots[].{ESTADO:State,DESCRIPCION:Description,NOMBRE:Tags[?Key=='Name'].Value | [0]}" --output table
```
