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
