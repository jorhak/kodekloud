```
The Nautilus DevOps team is expanding their AWS infrastructure and requires the setup of a private Virtual Private Cloud (VPC) along with a subnet. This VPC and subnet configuration will ensure that resources deployed within them remain isolated from external networks and can only communicate within the VPC. Additionally, the team needs to provision an EC2 instance under the newly created private VPC. This instance should be accessible only from within the VPC, allowing for secure communication and resource management within the AWS environment.

Create a VPC named nautilus-priv-vpc with the CIDR block 10.0.0.0/16.

Create a subnet named nautilus-priv-subnet inside the VPC with the CIDR block 10.0.1.0/24 and auto-assign IP option must not be enabled.

Create an EC2 instance named nautilus-priv-ec2 inside the subnet and instance type must be t2.micro.

Ensure the security group of the EC2 instance allows access only from within the VPC's CIDR block.

Create the main.tf file (do not create a separate .tf file) to provision the VPC, subnet and EC2 instance.

Use variables.tf file with the following variable names:

KKE_VPC_CIDR for the VPC CIDR block.
KKE_SUBNET_CIDR for the subnet CIDR block.
Use the outputs.tf file with the following variable names:

KKE_vpc_name for the name of the VPC.
KKE_subnet_name for the name of the subnet.
KKE_ec2_private for the name of the EC2 instance.

Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
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

# Crear variables.tf
```
vi variables.tf
```

```
variable "KKE_VPC_CIDR" {
  description = "Bloque CIDR VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "KKE_SUBNET_CIDR" {
  description = "Bloque CIDR SUBNET"
  type        = string
  default     = "10.0.1.0/24"
}
```
# Crear outputs.tf
```
vi outputs.tf
```

```
output "KKE_vpc_name" {
  description = "Nombre de la VPC private"
  value = aws_vpc.principal.tags.Name
}

output "KKE_subnet_name" {
  description = "Nombre de la SubNet private"
  value = aws_subnet.principal.tags.Name
}

output "KKE_ec2_private" {
  description = "Nombre de Instance private"
  value = aws_instance.servidor_privado.tags.Name
}
```
# Crear main.tf
```
vi main.tf
```

```
resource "aws_vpc" "principal" {
  cidr_block       = var.KKE_VPC_CIDR
  instance_tenancy = "default"

  tags = {
    Name = "xfusion-priv-vpc"
  }
}

resource "aws_subnet" "principal" {
  vpc_id     = aws_vpc.principal.id
  cidr_block = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false

  tags = {
    Name = "xfusion-priv-subnet"
  }
}

resource "aws_security_group" "permitir_red_interna" {
  name        = "permitir_trafico_vpc"
  description = "Perminte el acceso desde CIDR de la VPC"
  vpc_id      = aws_vpc.principal.id
  
  tags = {
    Name = "Acceso a CIDR de VPC"
  }
  
  ingress {
    description = "Permite trafico interno"
    from_port   = 0
    to_port     = 0
    protocol    = -1
    cidr_blocks = [aws_vpc.principal.cidr_block]
  }
  
  egress {
    description = "Conexion internet"
    from_port   = 0
    to_port     = 0
    protocol    = -1
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "servidor_privado" {
  ami           = "ami-0c101f26f147fa7fd"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.principal.id
  vpc_security_group_ids = [aws_security_group.permitir_red_interna.id]

  tags = {
    Name = "xfusion-priv-ec2"
  }
}
```
# Inicializar proyecto
```
terraform init
```
#### Validar codigo
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
aws ec2 describe-instances \
    --filters Name=tag:Name,Values="xfusion-priv-ec2"
```
