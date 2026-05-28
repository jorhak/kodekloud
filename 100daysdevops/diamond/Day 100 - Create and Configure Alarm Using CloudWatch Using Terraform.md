```
The Nautilus DevOps team has been tasked with setting up an EC2 instance for their application. To ensure the application performs optimally, they also need to create a CloudWatch alarm to monitor the instance's CPU utilization. The alarm should trigger if the CPU utilization exceeds 90% for one consecutive 5-minute period. To send notifications, use the SNS topic named datacenter-sns-topic, which is already created.

Launch EC2 Instance: Create an EC2 instance named datacenter-ec2 using any appropriate Ubuntu AMI (you can use AMI ami-0c02fb55956c7d316).

Create CloudWatch Alarm: Create a CloudWatch alarm named datacenter-alarm with the following specifications:

Statistic: Average
Metric: CPU Utilization
Threshold: >= 90% for 1 consecutive 5-minute period
Alarm Actions: Send a notification to the datacenter-sns-topic SNS topic.
Update the main.tf file (do not create a separate .tf file) to create a EC2 Instance and CloudWatch Alarm.

Create an outputs.tf file to output the following values:

KKE_instance_name for the EC2 instance name.
KKE_alarm_name for the CloudWatch alarm name.

Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
```
# 1 Crear terraform.tfvars
```
vi terraform.tfvars
```

```
SNS_NAME      = "devops-sns-topic"
INSTANCE_NAME = "devops-ec2"
IMAGE         = "ami-0c02fb55956c7d316"
CW_NAME       = "devops-alarm"
THRESHOLD     = 90
PERIOD        = 5
KEY_NAME      = "devops-key-pair"
```
# 2 Crear variables.tf
```
vi variables.tf
```

```
variable "SNS_NAME" {
  description = "Envia notificaciones"
  type        = string
}

variable "INSTANCE_NAME" {
  description = "Nombre de la instancia"
  type        = string
}

variable "IMAGE" {
  description = "ID de la imagen que vamos a utilizar"
  type        = string
}

variable "CW_NAME" {
  description = "Nombre de la alarma"
  type        = string
}

variable "THRESHOLD" {
  description = "Porcentaje de uso de la CPU"
  type        = number
}

variable "PERIOD" {
  description = "Perido para lanzar una alarma"
  type        = number
}

variable "KEY_NAME" {
  description = "Nombre de la llave"
  type        = string
}
```
# 3 Crear outputs.tf
```
vi outputs.tf
```

```
output "KKE_instance_name" {
  description = "Nombre de la instancia"
  value       = aws_instance.mi_instancia.tags.Name
}

output "KKE_alarm_name" {
  description = "Nombre de la alarma de CloudWatch"
  value       = aws_cloudwatch_metric_alarm.mi_alarma.alarm_name
}

output "ip_publica" {
  description = "IP pública de la instancia creada"
  value       = aws_instance.mi_instancia.public_ip
}

output "usuario" {
  description = "Nombre del usuario"
  value       = "ubuntu"
}
```
# 4 Crear main.tf
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
  key_name   = var.KEY_NAME
  public_key = tls_private_key.llave_privada.public_key_openssh
}

#Guardamos la llave privada en un fichero .pem
resource "local_file" "llave_pem" {
  content  = tls_private_key.llave_privada.private_key_pem
  filename = "${path.module}/datacenter-kp.pem"
  file_permission = "0600"
}

#Crear security group
resource "aws_security_group" "sg_datacenter" {
  name        = "sg_datacenter_ec2"
  description = "Security group explicito para evitar problemas de indexacion nula"
}

resource "aws_vpc_security_group_ingress_rule" "permitir_ssh" {
  security_group_id = aws_security_group.sg_datacenter.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 22
  ip_protocol = "tcp"
  to_port     = 22
}

#Creamos instancia
resource "aws_instance" "mi_instancia" {
  ami           = var.IMAGE
  instance_type = "t2.micro"
  key_name      = aws_key_pair.mi_llave_privada.key_name
  vpc_security_group_ids = [aws_security_group.sg_datacenter.id]
  associate_public_ip_address = true
  
  tags = {
    Name = var.INSTANCE_NAME
  }
}

#Verifica estado de instancia
resource "aws_ec2_instance_state" "test" {
  instance_id = aws_instance.mi_instancia.id
  state       = "running"
}

#Crear SNS
resource "aws_sns_topic" "notificacion" {
  name = var.SNS_NAME
}

#Crear subscripcion
resource "aws_sns_topic_subscription" "subcripcion" {
  topic_arn            = aws_sns_topic.notificacion.arn
  protocol             = "email"
  endpoint             = "hola@gmail.com"
}

resource "aws_cloudwatch_metric_alarm" "mi_alarma" {
  alarm_name                = var.CW_NAME
  alarm_description         = "Esta metrica monitoria el uso de cpu de la instancia"
  metric_name               = "CPUUtilization"
  namespace                 = "AWS/EC2"
  statistic                 = "Average"
  period                    = 300
  evaluation_periods        = 1
  threshold                 = 90
  comparison_operator       = "GreaterThanOrEqualToThreshold"
  alarm_actions       = [aws_sns_topic.notificacion.arn]
  ok_actions          = [aws_sns_topic.notificacion.arn]
  unit                = "Percent"
  dimensions          = {
    InstanceId        = aws_instance.mi_instancia.id
  }
}
```
# 5 Inicializar proyecto
```
terraform init
```
# 6 Validar codigo
```
terraform validate
```
# 7 Ejecutar
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
    --filters "Name=tag:Name,Values=devops-ec2" \
```

