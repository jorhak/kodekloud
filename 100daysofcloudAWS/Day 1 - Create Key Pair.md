```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a key pair with the following requirements:

- Name of the `key pair` should be `datacenter-kp`.
  
- Key pair `type` must be `rsa`
  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://503702267784.signin.aws.amazon.com/console?region=us-east-1](https://503702267784.signin.aws.amazon.com/console?region=us-east-1)|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Fri Jan 16 14:06:24 UTC 2026|
|End Time|Fri Jan 16 15:06:24 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Lo primero que vamos hacer es ver que llaves estan creadas:
```
aws ec2 describe-key-pairs
```

Ahora vamos a crear las llaves:
```
aws ec2 create-key-pair \
    --key-name "datacenter-kp" \
    --key-type "rsa" \
    --query "KeyMaterial" \
    --tag-specifications 'ResourceType=key-pair,Tags=[{Key=Desarrollo,Value=Dev},{Key=PreProduccion,Value=Staging}]' \
    --output text > mi-primer-llave.pem
```

Vamos a borrar las llaves
```
aws ec2 delete-key-pair --key-name "datacenter-kp-1"
```
