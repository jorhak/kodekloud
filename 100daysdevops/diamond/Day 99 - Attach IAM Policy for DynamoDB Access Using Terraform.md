```
The DevOps team has been tasked with creating a secure DynamoDB table and enforcing fine-grained access control using IAM. This setup will allow secure and restricted access to the table from trusted AWS services only.

As a member of the Nautilus DevOps Team, your task is to perform the following using Terraform:

Create a DynamoDB Table: Create a table named devops-table with minimal configuration.

Create an IAM Role: Create an IAM role named devops-role that will be allowed to access the table.

Create an IAM Policy: Create a policy named devops-readonly-policy that should grant read-only access (GetItem, Scan, Query) to the specific DynamoDB table and attach it to the role.

Create the main.tf file (do not create a separate .tf file) to provision the table, role, and policy.

Create the variables.tf file with the following variables:

KKE_TABLE_NAME: name of the DynamoDB table
KKE_ROLE_NAME: name of the IAM role
KKE_POLICY_NAME: name of the IAM policy
Create the outputs.tf file with the following outputs:

kke_dynamodb_table: name of the DynamoDB table
kke_iam_role_name: name of the IAM role
kke_iam_policy_name: name of the IAM policy
Define the actual values for these variables in the terraform.tfvars file.

Ensure that the IAM policy allows only read access and restricts it to the specific DynamoDB table created.


Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
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
variable "KKE_TABLE_NAME" {
  description = "DynamoDB table name"
  type        = string
}
variable "KKE_ROLE_NAME" {
  description = "IAM role name"
  type        = string
}
variable "KKE_POLICY_NAME" {
  description = "IAM policy"
  type        = string
}
```
# 3 Crear terraform.tfvars
```
vi terraform.tfvars
```

```
KKE_TABLE_NAME  = "datacenter-table"
KKE_ROLE_NAME   = "datacenter-role"
KKE_POLICY_NAME = "datacenter-readonly-policy"
```
# 4 Crear outputs.tf
```
vi outputs.tf
```

```
output "kke_dynamodb_table" {
  description = "DynamoDB table name"
  value       = aws_dynamodb_table.mi_base.name
}
output "kke_iam_role_name" {
  description = "IAM role name"
  value       = aws_iam_role.mi_rol.name
}
output "kke_iam_policy_name" {
  description = "IAM policy name"
  value       = aws_iam_policy.mi_politica.name
}

output "arn_policy" {
  description = "ARN Policy"
  value       = aws_iam_policy.mi_politica.arn
}
```
# 5 Crear main.tf
```
vi main.tf
```

```
resource "aws_dynamodb_table" "mi_base" {
  name             = var.KKE_TABLE_NAME
  billing_mode     = "PAY_PER_REQUEST"
  hash_key         = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

data "aws_iam_policy_document" "asumir_rol" {
  statement {
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }

    actions = ["sts:AssumeRole"]
  }
}

resource "aws_iam_role" "mi_rol" {
  name               = var.KKE_ROLE_NAME
  assume_role_policy = data.aws_iam_policy_document.asumir_rol.json
}

data "aws_iam_policy_document" "mi_politica" {
  statement {
    effect    = "Allow"
    actions   = [
      "dynamodb:GetItem",
      "dynamodb:Scan",
      "dynamodb:Query"
    ]
    resources = ["*"]
  }
}

resource "aws_iam_policy" "mi_politica" {
  name        = var.KKE_POLICY_NAME
  description = "Es una prueba de politica"
  policy      = data.aws_iam_policy_document.mi_politica.json
}

resource "aws_iam_policy_attachment" "adjuntar_politica" {
  name       = "Prueba adjuntar politica"
  roles      = [aws_iam_role.mi_rol.name]
  policy_arn = aws_iam_policy.mi_politica.arn
}
```
# 6 Inicializar proyecto
```
terraform init
```
# 7 Validar codigo
```
terraform validate
```
# 8 Ejecutar
#### Crear plan
```
terraform plan
```
#### Aplicar plan
```
terraform apply
```
# 9 Verificar
```
aws dynamodb describe-table \
    --table-name datacenter-table
```

```
aws iam get-role \
    --role-name datacenter-role
```

```
aws iam get-policy \
    --policy-arn "arn:aws:iam::000000000000:policy/datacenter-readonly-policy"
```

```
aws iam get-policy-version \
  --policy-arn "arn:aws:iam::000000000000:policy/datacenter-readonly-policy" \
  --version-id "v1" \
  --query "PolicyVersion.Document" \
  --output json
```