```
The Nautilus DevOps team needs to create an AWS Kinesis data stream for real-time data processing. This stream will be used to ingest and process large volumes of streaming data, which will then be consumed by various applications for analytics and real-time decision-making.

1. The stream should be named nautilus-stream.
2. Use Terraform to create this Kinesis stream.

The Terraform working directory is /home/bob/terraform. Create the main.tf file (do not create a different .tf file) to accomplish this task.

Note:
1. Right-click under the EXPLORER section in VS Code and select Open in
Integrated Terminal to launch the terminal.
2. Before submitting the task, ensure that terraform plan returns No changes.
Your infrastructure matches the configuration.
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
resource "aws_kinesis_stream" "test_kinesis" {
  name = "datacenter-stream"
  shard_count = 1
  stream_mode_details {
    stream_mode = "PROVISIONED"
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
```
aws kinesis list-streams
```

```
aws kinesis put-record \
    --stream-name datacenter-stream \
    --partition-key "clave-123" \
    --data "{\"mensaje\": \"Hola desde Pailon\!\"}"
```

```
ITERATOR=$(aws kinesis get-shard-iterator \
    --stream-name datacenter-stream \
    --shard-id "shardId-000000000000" \
    --shard-iterator-type TRIM_HORIZON\
    --query "ShardIterator" \
    --output text)
```

```
aws kinesis get-records --shard-iterator $ITERATOR \
| jq -r '.Records[].Data' \
| base64 --decode
```