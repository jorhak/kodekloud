```
The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

Create an IAM group named `iamgroup_mariyam`.  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://734081570291.signin.aws.amazon.com/console?region=us-east-1]|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Tue Mar 03 12:59:24 UTC 2026|
|End Time|Tue Mar 03 13:59:24 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Variables de entorno
```
IAM_GROUP_NAME=iamgroup_mariyam
REGION=us-east-1
```

1. Listar **IAM GROUPS**
```
aws iam list-groups
```

2. Crear **IAM GROUPS**
```
aws iam create-group \
    --group-name $IAM_GROUP_NAME \
    --region $REGION
```

3. Eliminar **IAM GROUPS**
Emplear solo si es necesario.
```
aws iam delete-group \
    --group-name $IAM_GROUP_NAME \
    --region $REGION
```

4. Verificar **IAM GROUPS**
```
aws iam get-group \
    --group-name $IAM_GROUP_NAME \
    --region $REGION
```
