```
As part of the data migration process, the Nautilus DevOps team is actively creating several S3 buckets on AWS. They plan to utilize both private and public S3 buckets to store the relevant data. Given the ongoing migration of other infrastructure to AWS, it is logical to consolidate data storage within the AWS environment as well.

1. Create a public S3 bucket named devops-s3-30368 using Terraform.
2. Ensure the bucket is accessible publicly once created by setting the proper ACL.
   
The Terraform working directory is /home/bob/terraform. Create the main.tf file (do not create a different .tf file) to accomplish this task.

Notes:
- Create the resources only in the us-east-1 region.
- Right-click under the EXPLORER section in VS Code and select Open in Integrated
  Terminal to launch the terminal.
- The name of the S3 bucket should be based on devops-s3-30368.
- You can use the ACL settings to ensure the bucket is publicly accessible.
```
# 1 Ver provider.tf
```
vi provider.tf
```

```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.91.0"
    }
  }
}

provider "aws" {
  region                      = "us-east-1"
  skip_credentials_validation = true
  skip_requesting_account_id  = true
  s3_use_path_style = true

endpoints {
    ec2            = "http://aws:4566"
    apigateway     = "http://aws:4566"
    cloudformation = "http://aws:4566"
    cloudwatch     = "http://aws:4566"
    dynamodb       = "http://aws:4566"
    es             = "http://aws:4566"
    firehose       = "http://aws:4566"
    iam            = "http://aws:4566"
    kinesis        = "http://aws:4566"
    lambda         = "http://aws:4566"
    route53        = "http://aws:4566"
    redshift       = "http://aws:4566"
    s3             = "http://aws:4566"
    secretsmanager = "http://aws:4566"
    ses            = "http://aws:4566"
    sns            = "http://aws:4566"
    sqs            = "http://aws:4566"
    ssm            = "http://aws:4566"
    stepfunctions  = "http://aws:4566"
    sts            = "http://aws:4566"
    rds            = "http://aws:4566"
  }
}
```
# 2 Crear main.tf
```
vi main.tf
```

```
resource "aws_s3_bucket" "primerbucket" {
  bucket = "xfusion-s3-19297"
}

resource "aws_s3_bucket_ownership_controls" "propietario" {
  bucket = aws_s3_bucket.primerbucket.id
  rule {
     object_ownership = "BucketOwnerPreferred"
  }
}

resource "aws_s3_bucket_public_access_block" "bloqueo" {
  bucket = aws_s3_bucket.primerbucket.id
  
  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_acl" "example" {
  depends_on = [
    aws_s3_bucket_ownership_controls.propietario,
    aws_s3_bucket_public_access_block.bloqueo,
  ]
  
  bucket = aws_s3_bucket.primerbucket.id
  acl    = "public-read"
}
```

# 3 Inicializar proyecto
```
terraform init
```
# 4 Validar codigo
```
terraform fmt
```

```
terraform validate
```
# 5 Ejecutar
#### Crear plan
```
terraform plan
```
#### Aplicar plan
```
terraform plan
```
# 6 Verificar
```
aws s3api get-bucket-acl \
    --bucket xfusion-s3-19297
```