```
The Nautilus DevOps team has been tasked with setting up an EC2 instance for their application. To ensure the application performs optimally, they also need to create a CloudWatch alarm to monitor the instance's CPU utilization. The alarm should trigger if the CPU utilization exceeds 90% for one consecutive 5-minute period. To send notifications, use the SNS topic named datacenter-sns-topic which is already created.

Launch EC2 Instance: Create an EC2 instance named datacenter-ec2 using any appropriate Ubuntu AMI.

Create CloudWatch Alarm: Create a CloudWatch alarm named datacenter-alarm with the following specifications:

Statistic: Average
Metric: CPU Utilization
Threshold: >= 90% for 1 consecutive 5-minute period.
Alarm Actions: Send a notification to datacenter-sns-topic.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://626885229782.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Mon Apr 27 18:19:26 UTC 2026
End Time	Mon Apr 27 19:19:26 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno
```
SNS_NAME=devops-sns-topic
INSTANCE_NAME=devops-ec2
CW_NAME=devops-alarm
INSTANCE_TYPE=t2.micro
REGION=us-east-1
IMAGE_ID=ami-0b6c6ebed2801a5cb
ALARM_NAME="HighCPUAlarm-Instance-$INSTANCE_NAME"
ALARM_DESCRIPTION="Alarm if CPU > 90% for 5 minutes"
METRIC_NAME=CPUUtilization
NAMESPACE=AWS/EC2
PERIOD=300
THRESHOLD=90
```
# 1 Crear EC2
```
INSTANCE_ID=$(aws ec2 run-instances \
    --image-id $IMAGE_ID \
    --instance-type $INSTANCE_TYPE \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$INSTANCE_NAME}]" \
    --region $REGION \
    --query "Instances[0].InstanceId" \
    --output text)    
```

```
aws ec2 wait instance-running \
    --instance-ids $INSTANCE_ID
```
### Estado de la instancia
```
aws ec2 describe-instances --instance-ids $INSTANCE_ID --query "Reservations[].Instances[].{Estado:State.Name,Nombre:Tags[].Value}" --output table
```
# 2 Configurar SNS
### Obtener arm de SNS
```
ARN_SNS=$(aws sns list-topics --query Topics[].TopicArn --output text)
```
### Suscriber un email
```
aws sns subscribe \
    --topic-arn $ARN_SNS \
    --protocol email \
    --notification-endpoint hola@gmail.com
```
### Publicar un mensaje
```
aws sns publish \
    --topic-arn $ARN_SNS \
    --message "Hello, this is a test notification from AWS CLI!" \
    --subject "Hola Pier"

```

```
aws sns list-subscriptions
```
# 3 Crear CloudWatch

```
aws cloudwatch put-metric-alarm \
    --alarm-name $CW_NAME \
    --alarm-description "$ALARM_DESCRIPTION" \
    --metric-name CPUUtilization \
    --namespace $NAMESPACE \
    --statistic Average \
    --period $PERIOD \
    --evaluation-periods 1 \
    --threshold $THRESHOLD \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=InstanceId,Value=$INSTANCE_ID \
    --evaluation-periods 1 \
    --alarm-actions $ARN_SNS \
    --unit Percent
```

``` 
aws cloudwatch describe-alarms --alarm-names $CW_NAME --query "MetricAlarms[*].AlarmName"

aws cloudwatch wait alarm-exists --alarm-names $CW_NAME
```
# Test
Ingresamos a la consola de AWS y realizamos las siguientes configuraciones habilitamos el puerto 22 y nos conectamos **Connect**>**EC2 Instance Connect**. Abrimos dos conexiones:
Conexion 1
```
sudo apt update
sudo apt install stress -y
stress --cpu 4 --timeout 400s
```

Conexion 2
```
htop
```
