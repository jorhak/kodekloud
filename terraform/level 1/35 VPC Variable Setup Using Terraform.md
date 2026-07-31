```
The Nautilus DevOps team is automating VPC creation using Terraform to manage networking efficiently. As part of this task, they need to create a VPC with specific requirements.

For this task, create an AWS VPC using `Terraform` with the following requirements:

1. The VPC name `devops-vpc` should be stored in a variable named `KKE_vpc`.
    
2. The VPC should have a CIDR block of `10.0.0.0/16`.
    

**Note:**

1. The configuration values should be stored in a `variables.tf` file.
    
2. The Terraform script should be structured with a `main.tf` file referencing `variables.tf`.
    
3. The Terraform working directory is `/home/bob/terraform`.
    
4. Right-click under the `EXPLORER` section in `VS Code` and select `Open in Integrated Terminal` to launch the terminal.
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

# 2 Crear variables.tf
```
vi variables.tf
```

```
variable "KKE_vpc" {
  description = "VPC con terraform"
  type        = string
  default     = "devops-vpc"
}

variable "KKE_cidr" {
  description = "Rango de IPs"
  type        = string
  default     = "10.0.0.0/16"
}

```
# 3 Crear main.tf
```
vi main.tf
```

```
resource "aws_vpc" "main" {
  cidr_block       = var.KKE_cidr

  tags = {
    Name = var.KKE_vpc
  }
}
```
# 4 Inicializar proyecto
```
terraform init
```
# 5 Validar codigo
```
terraform fmt
```

```
terraform validate
```
# 6 Ejecutar
#### Crear plan
```
terraform plan
```
#### Aplicar plan
```
terraform apply
```

# 7 Verificar
```
aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values="devops-vpc"
```