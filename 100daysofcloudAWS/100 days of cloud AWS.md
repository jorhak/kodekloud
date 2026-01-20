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
aws ec2 create-key-pair --key-name "datacenter-kp" --key-type "rsa" --query "KeyMaterial" --tag-specifications 'ResourceType=key-pair,Tags=[{Key=Desarrollo,Value=Dev},{Key=PreProduccion,Value=Staging}]' --output text > mi-primer-llave.pem
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



