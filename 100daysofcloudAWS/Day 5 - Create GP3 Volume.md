```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

Create a volume with the following requirements:

- Name of the volume should be `xfusion-volume`.
  
- Volume `type` must be `gp3`.
  
- Volume `size` must be `2 GiB`.
  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://919171157955.signin.aws.amazon.com/console?region=us-east-1](https://919171157955.signin.aws.amazon.com/console?region=us-east-1)|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Tue Jan 20 18:39:05 UTC 2026|
|End Time|Tue Jan 20 19:39:05 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Vamos a comenzar con la variables:
```
VOLUME_NAME=xfusion-volume
REGION=us-east-1
AZ=us-east-1a
```

Crear volumen:
```
aws ec2 create-volume \
    --availability-zone $AZ \
    --volume-type gp3 \
    --size 2 \
    --region $REGION \
    --tag-specifications "ResourceType=volume,Tags=[{Key=Name,Value='$VOLUME_NAME'}]"
```

Listar volumen creado:
```
aws ec2 describe-volumes
```
