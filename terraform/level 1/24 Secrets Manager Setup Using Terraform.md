```
The Nautilus DevOps team needs to store sensitive data securely using AWS Secrets Manager. They need to create a secret with the following specifications:

1) The secret name should be xfusion-secret.

2) The secret value should contain a key-value pair with username: admin and password: Namin123.

3) Use Terraform to create the secret in AWS Secrets Manager.

The Terraform working directory is /home/bob/terraform. Create the main.tf file (do not create a different .tf file) to accomplish this task.

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
resource "aws_secretsmanager_secret" "secreto" {
  name = "xfusion-secret"
}

variable "variables" {
  default = {
    username = "admin"
    password = "Namin123"
  }
  type = map(string)
}

resource "aws_secretsmanager_secret_version" "valores" {
  secret_id     = aws_secretsmanager_secret.secreto.id
  secret_string = jsonencode(var.variables)
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
aws secretsmanager list-secrets \
    --filters Key=name,Values=xfusion-secret
```

```
aws secretsmanager describe-secret \
    --secret-id xfusion-secret
```

```
aws secretsmanager get-secret-value \
    --secret-id xfusion-secret
```
En describe-secret: --secret-id se debe colocar el nombre del secreto o ARN.