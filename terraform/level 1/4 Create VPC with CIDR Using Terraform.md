```
The Nautilus DevOps team is strategically planning the migration of a portion of their infrastructure to the
AWS cloud. Acknowledging the magnitude of this endeavor, they have chosen to tackle the migration
incrementally rather than as a single, massive transition. Their approach involves creating Virtual Private
Clouds (VPCs) as the initial step, as they will be provisioning various services under different VPCs.
Create a VPC named devops-vpc in us-east-1 region with 192.168.0.0/24 IPv4 CIDR using
terraform.
The Terraform working directory is /home/bob/terraform. Create the main.tf file (do not create
a different .tf file) to accomplish this task.
Note: Right-click under the EXPLORER section in VS Code and select Open in Integrated
Terminal to launch the terminal.
```
# Ver provider.tf
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
  region = "us-east-1"
  skip_credentials_validation = true
  skip_requesting_account_id = true
  s3_use_path_style = true
  
endpoints {
    ec2 = "http://aws:4566"
    apigateway = "http://aws:4566"
    cloudformation = " http://aws:4566"
    cloudwatch = "http://aws:4566"
    dynamodb = "http://aws:4566"
    es = "http://aws:4566"
    firehose = "http://aws:4566"
    iam = "http://aws:4566"
    kinesis = "http://aws:4566"
    lambda = "http://aws:4566"
    route53 = "http://aws:4566"
    redshift = "http://aws:4566"
    s3 = "http://aws:4566"
    secretsmanager = "http://aws:4566"
    ses = "http://aws:4566"
    sns = "http://aws:4566"
    sqs = "http://aws:4566"
    ssm = "http://aws:4566"
    stepfunctions = "http://aws:4566"
    sts = "http://aws:4566"
    rds = "http://aws:4566"
  }
}
```
# Crear main.tf
```
vi main.tf
```

```
resource "aws_vpc" "main" {
  cidr_block = " 192.168.0.0/24"

  tags = {
    Name= "devops-vpc"
  }
}
```
# Inicializar proyecto
```
terraform init
```
# Validar codigo
```
terraform validate
```
# Ejecutar
#### Crear plan
```
terraform plan
```
#### Aplicar plan
```
terraform apply
```
# Verificar
```
aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=devops-vpc
```