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
