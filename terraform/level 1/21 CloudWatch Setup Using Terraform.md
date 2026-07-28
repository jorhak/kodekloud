```
The Nautilus DevOps team needs to set up CloudWatch logging for their application. They need to create a CloudWatch log group and log stream with the following specifications:

1) The log group name should be devops-log-group.

2) The log stream name should be devops-log-stream.

Use Terraform to create the CloudWatch log group and log stream. The Terraform working directory is /home/bob/terraform. Create the main.tf file (do not create a different .tf file) to accomplish this task.

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
resource "aws_cloudwatch_log_group" "yada" {
  name = "devops-log-group"
  tags = {
    Environment = "production"
    Application = "serviceA"
  }
}

resource "aws_cloudwatch_log_stream" "foo" {
  name           = "devops-log-stream"
  log_group_name = aws_cloudwatch_log_group.yada.name

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
```
aws logs describe-log-groups \
  --log-group-name-prefix "devops-log-group" \
  --query "logGroups[?logGroupName=='devops-log-group']"
```

```
aws logs describe-log-streams \
  --log-group-name "devops-log-group" \
  --log-stream-name-prefix "devops-log-stream" \
  --query "logStreams[?logStreamName=='devops-log-stream']"
```