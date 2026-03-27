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
