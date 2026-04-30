```
The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

An instance named devops-ec2 and a volume named devops-volume already exists in us-east-1 region. Attach the devops-volume volume to the devops-ec2 instance, make sure to set the device name to /dev/sdb while attaching the volume.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://644306594621.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
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
