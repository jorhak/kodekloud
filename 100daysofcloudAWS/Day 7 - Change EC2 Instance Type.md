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
