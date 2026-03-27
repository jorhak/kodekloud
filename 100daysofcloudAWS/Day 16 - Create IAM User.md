```
When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements:

For this task, create an IAM user named `iamuser_ravi`.  
  

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://745131131958.signin.aws.amazon.com/console?region=us-east-1]|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Mon Mar 02 18:03:18 UTC 2026|
|End Time|Mon Mar 02 19:03:18 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Variables de entorno:
```
IAM_NAME=iamuser_ravi
REGION=us-east-1
```

1. Crear **IAM**:
```
aws iam create-user \
    --user-name $IAM_NAME \
    --region $REGION 
```

2. Verificar creacion de **IAM**:
```
aws iam list-users
```
