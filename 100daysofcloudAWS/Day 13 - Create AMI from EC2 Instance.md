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