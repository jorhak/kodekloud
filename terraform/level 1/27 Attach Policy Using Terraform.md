```
The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

An IAM user named iamuser_rose and a policy named iampolicy_rose already exists. Use Terraform to attach the IAM policy iampolicy_rose to the IAM user iamuser_rose. The Terraform working directory is /home/bob/terraform. Update the main.tf file (do not create a separate .tf file) to attach the specified IAM policy to the IAM user.

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
# Create IAM user
resource "aws_iam_user" "user" {
  name = "iamuser_rose"
  tags = {
    Name = "iamuser_rose"
  }
}

# Create IAM Policy
resource "aws_iam_policy" "policy" {
  name        = "iampolicy_rose"
  description = "IAM policy allowing EC2 read actions for rose"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["ec2:Read*"]
        Resource = "*"
      }
    ]
  })
}

resource "aws_iam_user_policy_attachment" "test-attach" {
  user       = aws_iam_user.user.name
  policy_arn = aws_iam_policy.policy.arn
}
```
# 3 Validar codigo
```
terraform validate
```
# 4 Ejecutar
#### Crear plan
```
terraform plan
```
#### Aplicar plan
```
terraform apply
```
# 5 Verificar
```
aws iam list-attached-user-policies \
    --user-name iamuser_rose
```

```
aws iam get-user \
    --user-name iamuser_rose
```

```
aws iam get-policy-version \
    --version-id v1 \
    --policy-arn arn:aws:iam::000000000000:policy/iampolicy_rose
```

```
aws iam list-policies \
    --query "Policies[].{Nombre:PolicyName,Arn:Arn}" \
    --output table | grep iampolicy_rose
```

```
aws iam get-policy \
    --policy-arn arn:aws:iam::000000000000:policy/iampolicy_rose
```