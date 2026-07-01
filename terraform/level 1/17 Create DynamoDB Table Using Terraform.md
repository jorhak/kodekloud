```
The Nautilus DevOps team needs to set up a DynamoDB table for storing user data. They need to create a DynamoDB table with the following specifications:

1) The table name should be `datacenter-users`.

2) The primary key should be `datacenter_id` (String).

3) The table should use `PAY_PER_REQUEST` billing mode.

Use `Terraform` to create this `DynamoDB table`. The Terraform working directory is `/home/bob/terraform`. Create the `main.tf` file (do not create a different `.tf` file) to create the DynamoDB table.

`Note:` Right-click under the `EXPLORER` section in `VS Code` and select `Open in Integrated Terminal` to launch the terminal.
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
resource "aws_dynamodb_table" "primertabla" {
  name           = "datacenter-users"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "datacenter_id"

  attribute {
    name = "datacenter_id"
    type = "S"
  }

  tags = {
    Name        = "dynamodb-test"
    Environment = "production"
  }
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
terraform apply
```

# 6 Verificar
#### Validar tabla
```
aws dynamodb describe-table \
    --table-name datacenter-users
```
#### Ingresar datos
```
aws dynamodb put-item \
    --table-name datacenter-users \
    --item '{
        "datacenter_id": {"S": "dc-us-east-01"},
        "nombre_usuario": {"S": "Bob_DevOps"},
        "rol": {"S": "Admin"},
        "activo": {"BOOL": true}
    }'
```
#### Mostrar datos
```
aws dynamodb scan \
    --table-name datacenter-users
```
