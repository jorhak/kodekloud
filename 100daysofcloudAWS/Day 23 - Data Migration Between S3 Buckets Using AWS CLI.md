```
As part of a data migration project, the team lead has tasked the team with migrating data from an existing S3 bucket to a new S3 bucket. The existing bucket contains a substantial amount of data that must be accurately transferred to the new bucket. The team is responsible for creating the new S3 bucket and ensuring that all data from the existing bucket is copied or synced to the new bucket completely and accurately. It is imperative to perform thorough verification steps to confirm that all data has been successfully transferred to the new bucket without any loss or corruption.

As a member of the Nautilus DevOps Team, your task is to perform the following:

Create a New Private S3 Bucket: Name the bucket xfusion-sync-15425.

Data Migration: Migrate the entire data from the existing xfusion-s3-3756 bucket to the new xfusion-sync-15425 bucket.

Ensure Data Consistency: Ensure that both buckets have the same data.

Use AWS CLI: Use the AWS CLI to perform the creation and data migration tasks.



Notes:

Create the resources only in us-east-1 region.
To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:\ntoggle button
```

Variables de entorno:
```
S3_NAME=xfusion-sync-15425
S3_SOURCE_NAME=xfusion-s3-3756
REGION=us-east-1
```

1. Crear _bucket_
```
aws s3api create-bucket \
    --acl private \
    --bucket $S3_NAME \
    --region $REGION
```

2. Verificar creacion de _bucket_
```
aws s3api list-buckets --query "Buckets[].Name"
```

3. Listar el contenido del _bucket_
```
aws s3 ls s3://$S3_SOURCE_NAME
```

4. Copiar del bucket **S3_SOURCE_NAME** a **S3_NAME**
```
aws s3 sync s3://$S3_SOURCE_NAME s3://$S3_NAME
```

5. Verificar que se copiaron los ficheros
```
aws s3 ls s3://$S3_NAME
```
