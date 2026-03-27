```
The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

An IAM user named iamuser_rose and a policy named iampolicy_rose already exist. Attach the IAM policy iampolicy_rose to the IAM user iamuser_rose.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://637423303501.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Fri Mar 06 14:15:43 UTC 2026
End Time	Fri Mar 06 15:15:43 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
IAM_USER_NAME=iamuser_rose
IAM_POLICY_NAME=iampolicy_rose
REGION=us-east-1
```

1. Obtener _policy-arn_ de la **policy**:
```
aws iam list-policies \
    --region $REGION \
    --max-items 4
```


2. Adjuntar **policy**
```
aws iam attach-user-policy \
--user-name $IAM_USER_NAME \
--policy-arn "arn:aws:iam::637423303501:policy/iampolicy_rose"
```

3. Verificar
```
aws iam get-user \
    --user-name $IAM_USER_NAME
```

```
aws iam list-attached-user-policies \
    --user-name $IAM_USER_NAME
```
