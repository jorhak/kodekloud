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
