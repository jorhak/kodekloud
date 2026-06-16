```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS
cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in
incremental steps rather than as a single massive transition. To achieve this, they have segmented large
tasks into smaller, more manageable units.
For this task, create an EC2 instance using Terraform with the following requirements:
1. The EC2 instance must use the value nautilus-ec2 as its Name tag, which defines the
instance name in AWS.
2. Use the Amazon Linux ami-0c101f26f147fa7fd to launch this instance.
3. The Instance type must be t2.micro.
4. Create a new RSA key named nautilus-kp.
5. Attach the default (available by default) security group.
The Terraform working directory is /home/bob/terraform. Create the main.tf file (do not
create a different .tf file) to provision the instance.
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
#Creamos el par de llaves
resource "tls_private_key" "llave_privada" {
  algorithm = "RSA"
  rsa_bits = 4096
}

#Creamos la llave en AWS
resource "aws_key_pair" "mi_llave_publica" {
  key_name = "nautilus-kp"
  public_key =
  tls_private_key.llave_privada.public_key_openssh
}

#Guardamos la llave privada en un fichero .pem
resource "local_file" "llave_pem" {
  content = tls_private_key.llave_privada.private_key_pem
  filename = "${path.module}/devops-kp.pem"
  file_permission = "0600"
}

resource "aws_instance" "example" {
  ami = "ami-0c101f26f147fa7fd"
  instance_type = "t2.micro"
  key_name = aws_key_pair.mi_llave_publica.key_name

  tags = {
    Name= "nautilus-ec2"
  }
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
aws ec2 describe-instances \
    --filters Name=tag:Name,Values=nautilus-ec2 \
    --query "Reservations[*].Instances[*].{Nombre:Tags[0].Value,Estado:State.Name,Llave:KeyName,IPPublica:PublicIpAddress,IPPrivada:PrivateIpAddress}" \
    --output table
```