```bash
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. As part of this phased migration approach, they need to allocate an Elastic IP address to support external access for specific workloads.

For this task, create an AWS Elastic IP using Terraform with the following requirement:

The Elastic IP name nautilus-eip should be stored in a variable named KKE_eip. The Terraform working directory is /home/bob/terraform.

Note:
The configuration values should be stored in a variables.tf file.

The Terraform script should be structured with a main.tf file referencing variables.tf.

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
variable "KKE_eip" {
  description = "Elastic IP"
  type        = string
  default     = "nautilus-eip" 
}
EOF
```
# 3 Crear main.tf
```bash
cat << EOF > main.tf
resource "aws_eip" "ipam-ip" {
  domain       = "vpc"
  
  tags = {
    Name = var.KKE_eip
  }
}
EOF
```
# 3 Inicializar proyecto
```bash
terraform init
```
# 4 Validar codigo
```bash
terraform fmt
terraform validate
```
# 5 Ejecutar
#### Crear plan
```bash
terraform plan
```
#### Aplicar plan
```bash
terraform apply
```

# 6 Verificar
```bash
aws ec2 describe-addresses \
    --filters Name=tag:Name,Values=nautilus-eip
```
#### Crear EIP (No Ejecutar)
```bash
aws ec2 allocate-address \
    --domain vpc \
    --tag-specifications "ResourceType=elastic-ip,Tags=[{Key=Name,Value=devops-eip}]"
```