```bash
The Nautilus DevOps team is automating IAM user creation using Terraform for better identity management.

For this task, create an AWS IAM User using Terraform with the following requirements:

The IAM User name iamuser_rose should be stored in a variable named KKE_user.

Note:
1. The configuration values should be stored in a variables.tf file.
2. The Terraform script should be structured with a main.tf file referencing variables.tf.

The Terraform working directory is /home/bob/terraform.
Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.
```
# 1 Ver provider.tf
```bash
vi provider.tf
```

```bash
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
```bash
cat << EOF > variables.tf
variable "KKE_user" {
  description = "IAM usuario"
  type        = string
  default     = "iamuser_rose"
}
EOF
```
# 3 Crear main.tf
```bash
cat << EOF > main.tf
resource "aws_iam_user" "lb" {
  name = var.KKE_user
  
  tags = {
    Env = "DEV"
  }
}
EOF
```
# 4 Inicializar proyecto
```bash
terraform init
```
# 5 Validar codigo
```bash
terraform fmt
terraform validate
```
# 6 Ejecutar
#### Crear plan
```bash
terraform plan
```
#### Aplicar plan
```bash
terraform apply
```

# 7 Verificar
```bash
aws iam get-user \
    --user-name iamuser_rose
```