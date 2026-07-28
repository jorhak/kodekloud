```
The Nautilus DevOps team is currently engaged in a cleanup process, focusing on removing unnecessary data and services from their AWS account. As part of the migration process, several resources were created for one-time use only, necessitating a cleanup effort to optimize their AWS environment.

A S3 bucket named devops-bck-22915 already exists.

1) Copy the contents of devops-bck-22915 S3 bucket to /opt/s3-backup/ directory on terraform-client host (the landing host once you load this lab).

2) Delete the S3 bucket devops-bck-22915.

3) Use the AWS CLI through Terraform to accomplish this task—for example, by running AWS CLI commands within Terraform. The Terraform working directory is /home/bob/terraform. Update the main.tf file (do not create a separate .tf file) to accomplish this task.

Note: Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.
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
# Variables de entorno
```
PREFIX=devops
S3_NAME=$PREFIX-bck-22915
DEST=/opt/s3-backup/
```
# Descargar
```
aws s3 sync s3://$S3_NAME $DEST
ls $DEST
```
# 2 Crear main.tf
```
vi main.tf
```

```
resource "null_resource" "s3_backup_and_destroy" {
  #Asugura que el directorio existe
  provisioner "local-exec" {
    command = "mkdir -p /opt/s3-backup/"
  }
  #Descarga el contenido
  provisioner "local-exec" {
    command = "aws s3 sync s3://devops-bck-22915 /opt/s3-backup/"
  }
  #Elimina Bucket
  provisioner "local-exec" {
    command = "aws s3 rb s3://devops-bck-22915 --force"
  }
}
```
# 3 Inicializar proyecto
```
terraform init
```
# 4 Validar codigo
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
terraform apply
```

# 6 Verificar
```
aws s3api list-buckets \
    --prefix devops
```