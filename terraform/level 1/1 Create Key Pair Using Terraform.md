```
The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a key pair using Terraform with the following requirements:

Name of the key pair should be devops-kp.

Key pair type must be rsa.

The private key file should be saved under /home/bob/devops-kp.pem.
The Terraform working directory is /home/bob/terraform. Create the main.tf file (do not create a different .tf file) to accomplish this task.

Note: Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.
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
  rsa_bits  = 4096
}

#Creamos la llave en AWS
resource "aws_key_pair" "mi_llave_privada" {
  key_name   = "devops-kp"
  public_key = tls_private_key.llave_privada.public_key_openssh
}

#Guardamos la llave privada en un fichero .pem
resource "local_file" "llave_pem" {
  content  = tls_private_key.llave_privada.private_key_pem
  filename = "../${path.module}/devops-kp.pem"
  file_permission = "0600"
}
```
# Inicializar proyecto
```
terraform init
```
# Verificar Codigo
```
terraform validate
```
# Ejecutar
#### Crear plan
```
terraform plan
```
#### Ejecutar plan
```
terraform apply
```
# Verificar que se creo la llave en AWS
```
aws ec2 describe-key-pairs \
    --key-names devops-kp
```