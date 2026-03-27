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
