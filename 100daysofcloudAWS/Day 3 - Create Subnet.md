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
