```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

1. For this task, create an AMI from an existing EC2 instance named datacenter ec2 using Terraform.
2. Name of the AMI should be datacenter-ec2-ami, make sure AMI is in available state.
3. The Terraform working directory is /home/bob/terraform. Update the main.tf file (do not create a separate .tf file) to create the AMI.

Note: Right-click under the EXPLORER section in VS Code and select Open in
Integrated Terminal to launch the terminal.
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
# Crear main.tf
```
vi main.tf
```

```
# Provision EC2 instance
resource "aws_instance" "ec2" {
  ami             = "ami-0c101f26f147fa7fd"
  instance_type   = "t2.micro"
  vpc_security_group_ids = [
    "sg-43cefae3be3dd0c54"
  ]

  tags = {
    Name= "datacenter-ec2"
  }
}

resource "aws_ami_from_instance" "mi_imagen" {
  name = "datacenter-ec2-ami"
  source_instance_id = aws_instance.ec2.id
}

output "id_imagen" {
  value = aws_ami_from_instance.mi_imagen.id
  description = "Id de mi nueva imagen"
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
aws ec2 describe-images \
    --image-ids ami-5b3caafb40c596bf8 \
    --query "Images[0].{Nombre:Name,Estado:State}" \
    --output table
```