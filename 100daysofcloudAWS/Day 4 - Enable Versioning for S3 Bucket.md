```
Data protection and recovery are fundamental aspects of data management. It's essential to have systems in place to ensure that data can be recovered in case of accidental deletion or corruption. The DevOps team has received a requirement for implementing such measures for one of the S3 buckets they are managing.

The s3 bucket name is datacenter-s3-12881, enable versioning for this bucket.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://995698815753.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Tue Jan 20 17:27:50 UTC 2026
End Time	Tue Jan 20 18:27:50 UTC 2026

Notes:

Create the resources only in us-east-1 region.
To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:\ntoggle button
```

Vamos a comenzar creando las variables:
```
S3_NAME=datacenter-s3-12881
REGION=us-east-1
```

Crear **Bucket**:
```
aws s3api put-bucket-versioning \
    --bucket $S3_NAME \
    --region $REGION \
    --versioning-configuration Status=Enabled
```

Verificar que se creo el **Bucket**
```
aws s3api list-buckets
```
