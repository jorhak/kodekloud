```
The Nautilus DevOps team is tasked with deploying a containerized application using Amazon's container services. They need to create a private Amazon Elastic Container Registry (ECR) to store their Docker images and use Amazon Elastic Container Service (ECS) to deploy the application. The process involves building a Docker image from a given Dockerfile, pushing it to the ECR, and then setting up an ECS cluster to run the application.

Create a Private ECR Repository:

Create a private ECR repository named nautilus-ecr to store Docker images.
Build and Push Docker Image:

Use the Dockerfile located at /root/pyapp on the aws-client host.
Build a Docker image using this Dockerfile.
Tag the image with latest tag.
Push the Docker image to the nautilus-ecr repository.
Create and Configure ECS cluster:

Create an ECS cluster named nautilus-cluster using the Fargate launch type.
Create an ECS Task Definition:

Define a task named nautilus-taskdefinition using the Docker image from the nautilus-ecr ECR repository.
Specify necessary CPU and memory resources.
Deploy the Application Using ECS Service:

Create a service named nautilus-service on the nautilus-cluster to run the task.
Ensure the service runs at least one task.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://604120469714.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Wed Aug 12 13:33:28 UTC 2026
End Time	Wed Aug 12 14:33:28 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
ACCOUNT_ID=$(aws account get-account-information \
    --query AccountId \
    --output text)
PREFIX=datacenter
ECR_NAME=$PREFIX-ecr
TAG=latest
ECS_NAME=$PREFIX-cluster
DEFINITION_NAME=$PREFIX-taskdefinition
SERVICE_NAME=$PREFIX-service
REGION=us-east-1
URI="${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/${ECR_NAME}"
```
# Iniciar session en docker
```
aws ecr get-login-password \
    --region $REGION \
    | docker login \
    --username AWS \
    --password-stdin "${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com"
```
# 1 Crear repositorio privado ECR
```
aws ecr create-repository \
    --repository-name $ECR_NAME \
    --tags Key=ENV,Value=DEV \
    --region $REGION
```
De aqui tambien podemos obtener la **URI** del parametro **respositoryUri**.
# 2 Crear Imagen
```
cd /root/pyapp
docker build --platform linux/amd64 --no-cache -t $ECR_NAME:$TAG .
```
## 2.1 Subir imagen al repositorio
```
docker tag $ECR_NAME:$TAG $URI:$TAG
docker push $URI:$TAG
```
# 3 Crear ECS Cluster en modo Fargate
```
aws ecs create-cluster \
    --cluster-name $ECS_NAME \
    --tags key=ENV,value=DEV \
    --region $REGION
```
# 4 Crear Role
## 4.1 Crear el archivo de Trust Policy para ECS
```
EXECUTION_ROLE_NAME="ecsTaskExecutionRole"

cat <<EOF > ecs-trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```
## 4.2 Crear el rol en IAM
```
aws iam create-role \
    --role-name $EXECUTION_ROLE_NAME \
    --assume-role-policy-document file://ecs-trust-policy.json 2>/dev/null || true
```
## 4.3 Adjuntar la política oficial de AWS para ejecución de tareas de ECS
```
aws iam attach-role-policy \
    --role-name $EXECUTION_ROLE_NAME \
    --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy 2>/dev/null || true
```
## 4.4 Obtener el ARN del rol creado
```
EXEC_ROLE_ARN=$(aws iam get-role --role-name $EXECUTION_ROLE_NAME --query "Role.Arn" --output text)

echo "Execution Role ARN: $EXEC_ROLE_ARN"
```
# 5 Crear Task Definition
```
cat <<EOF > task-definition.json
{
  "family": "${DEFINITION_NAME}",
  "networkMode": "awsvpc",
  "executionRoleArn": "${EXEC_ROLE_ARN}",
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "pyapp-container",
      "image": "${URI}:${TAG}",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp"
        }
      ]
    }
  ]
}
EOF
```

```
aws ecs register-task-definition \
    --cli-input-json file://task-definition.json \
    --region $REGION
```
# 6 Crear Service
## 6.1 Antes debemos obtener una Subnet y un Security Group de la VPC por defecto
```
DEFAULT_VPC=$(aws ec2 describe-vpcs \
    --filters "Name=isDefault,Values=true" \
    --query "Vpcs[0].VpcId" \
    --output text)

SUBNET_ID=$(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=${DEFAULT_VPC}" \
    --query "Subnets[0].SubnetId" \
    --output text)

SECURITY_GROUP_ID=$(aws ec2 describe-security-groups \
    --filters "Name=vpc-id,Values=${DEFAULT_VPC}" "Name=group-name,Values=default" \
    --query "SecurityGroups[0].GroupId" \
    --output text)
```
## 6.2 Habilitar permisos para HTTP
```
aws ec2 authorize-security-group-ingress \
    --group-id $SECURITY_GROUP_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0 \
    --region $REGION 2>/dev/null || true
```
## 6.3 Crear servicio ECS
```
aws ecs create-service \
    --cluster $ECS_NAME \
    --service-name $SERVICE_NAME \
    --task-definition $DEFINITION_NAME \
    --desired-count 1 \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[${SUBNET_ID}],securityGroups=[${SECURITY_GROUP_ID}],assignPublicIp=ENABLED}" \
    --region $REGION
    
sleep 15
```
# 7 Obtener IP publica
## 7.1 Obtener el ARN de la tarea en ejecución
```
TASK_ARN=$(aws ecs list-tasks \
    --cluster $ECS_NAME \
    --region $REGION \
    --query "taskArns[0]" \
    --output text)

echo "Task ARN: $TASK_ARN"
```
## 7.2 Obtenemos el ID de la interfaz de red (ENI) de la tarea
```
ENI_ID=$(aws ecs describe-tasks \
    --cluster $ECS_NAME \
    --tasks "$TASK_ARN" \
    --region $REGION \
    --query "tasks[0].attachments[0].details[?name=='networkInterfaceId'].value" \
    --output text)
```
## 7.3 Consultamos la IP pública vinculada a esa ENI
```
PUBLIC_IP=$(aws ec2 describe-network-interfaces \
    --network-interface-ids "$ENI_ID" \
    --region $REGION \
    --query "NetworkInterfaces[0].Association.PublicIp" \
    --output text) 
    
echo "IP Pública de tu App: $PUBLIC_IP"
```
# 8 Verificacion
Ver que el servcio este corriendo
```
aws ecs list-tasks --cluster $ECS_NAME --region $REGION
```

Ver que la imagen se subio
```
aws ecr list-images \
    --repository-name $ECR_NAME \
    --region $REGION
    
aws ecr describe-images \
    --repository-name $ECR_NAME \
    --region $REGION
```

Ver la conexion por curl
```
curl http://$PUBLIC_IP
```





