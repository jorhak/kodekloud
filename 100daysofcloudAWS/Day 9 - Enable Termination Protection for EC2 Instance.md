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
