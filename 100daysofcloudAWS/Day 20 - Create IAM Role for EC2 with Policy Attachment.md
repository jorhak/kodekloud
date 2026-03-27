```
When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements:

Create an IAM role as below:

1) IAM role name must be `iamrole_jim`.

2) Entity type must be `AWS Service` and use case must be `EC2`.

3) Attach a policy named `iampolicy_jim`.  
  

  

Use the below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://453274592256.signin.aws.amazon.com/console?region=us-east-1]|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Sat Mar 07 01:07:19 UTC 2026|
|End Time|Sat Mar 07 02:07:19 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```

Variables de entorno:
```
REGION=us-east-1
IAM_ROLE_NAME=iamrole_jim
POLICY_NAME=iampolicy_jim
```

1. Crear **IAM Role**
```code
vi policy.json
```

```json
{
  "Version":"2012-10-17",		 	 	 
  "Statement": {
    "Effect": "Allow",
    "Principal": {
	    "Service": "ec2.amazonaws.com"
	},
    "Action": "sts:AssumeRole"
  }
}

```

```code
aws iam create-role \
    --role-name $IAM_ROLE_NAME \
    --assume-role-policy-document file://policy.json \
    --region $REGION
```
 
2. Adjuntar **Policy**
- Listar y obtener **policy-arn**
```code
aws iam list-policies \
    --max-items 4 \
    --region $REGION
```
- Adjuntar **Policy**
```code
aws iam attach-role-policy \
    --role-name $IAM_ROLE_NAME \
    --policy-arn "arn:aws:iam::453274592256:policy/iampolicy_jim" \
    --region $REGION
```

3. Verificar
```code
aws iam get-role \
    --role-name $IAM_ROLE_NAME \
    --query "Role.[RoleName, Arn]" \
    --output table
```

``` code
aws iam list-attached-role-policies \
    --role-name $IAM_ROLE_NAME
```
