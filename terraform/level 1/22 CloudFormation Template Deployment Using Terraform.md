```
The Nautilus DevOps team is working on automating infrastructure deployment using AWS CloudFormation. As part of this effort, they need to create a CloudFormation stack that provisions an S3 bucket with versioning enabled.

Create a CloudFormation stack named datacenter-stack using Terraform. This stack should contain an S3 bucket named datacenter-bucket-22698 as a resource, and the bucket must have versioning enabled. The Terraform working directory is /home/bob/terraform. Create the main.tf file (do not create a different .tf file) to accomplish this task.

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

# 2 Crear main.tf
```
vi main.tf
```

```
resource "aws_cloudformation_stack" "datacenter_stack" {
  name = "datacenter-stack"
  # Aquí definimos la plantilla de CloudFormation embebida en formato YAML
  template_body = <<STACK
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Stack que aprovisiona un bucket S3 con versionamiento habilitado'
Resources:
  DatacenterBucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: 'datacenter-bucket-22698'
      VersioningConfiguration:
        Status: Enabled
STACK
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
    --prefix datacenter
```

```
aws cloudformation describe-stacks \
    --stack-name "datacenter-stack"
```

```
aws cloudformation describe-stack-resources \
    --stack-name "datacenter-stack"
```