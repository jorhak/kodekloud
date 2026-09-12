```
The Nautilus DevOps team needs to implement a Lambda function using a CloudFormation stack. Create a CloudFormation template named /root/devops-lambda.yml on the AWS client host and configure it to create the following components. The stack name must be devops-lambda-app.

Create a Lambda function named devops-lambda. Set the function memory to 128 MB and the timeout to 10 seconds.
Use the Runtime Python.
The function should print the body Welcome to KKE AWS Labs!.
Ensure the status code is 200.
Create and use the IAM role named lambda_execution_role.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://433114257483.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Sat Sep 12 02:27:47 UTC 2026
End Time	Sat Sep 12 03:27:47 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```bash
PREFIX=datacenter
CF_TEMPLATE=/root/$PREFIX-lambda.yml
STACK_NAME=$PREFIX-lambda-app
LAMBDA_NAME=$PREFIX-lambda
RUNTIME=python3.12
MEMORY=128
TIMEOUT=10
MESSAGE="Welcome to KKE AWS Labs!"
ROLE_NAME=lambda_execution_role
REGION=us-east-1
```
# 1 Crear Lambda Funtion
```bash
vi index.py
```

```python
import os
from flask import Flask, jsonify, request
from mangum import Mangum

app = Flask(__name__)

@app.route('/', methods=['GET'])
def index():
    # Lee la variable de entorno. Si no existe, usa un mensaje por defecto. 
    welcome_message = os.environ.get("WELCOME_MSG", "Welcome to KKE AWS Labs!")
    return jsonify({"message":welcome_message}), 200
    
# 1. Creamos el adaptador de Mangum    
asgi_handler = Mangum(app)

def lambda_handler(event, context): 
    # Mangum se encarga de procesar el evento y el contexto de AWS 
    return asgi_handler(event, context)
```
# 1.1 Probarlo en local
#### 1.1.1 Crear pyproject.toml
```python
vi pyproject.toml
```

```python
[project]
name = "kke-aws-labs-api"
version = "0.1.0"
description = "Flask API para AWS Lambda configurada con variables de entorno"
requires-python = ">=3.12"
dependencies = [
    "flask>=3.0.3",
    "mangum>=0.17.0",
]

[tool.uv]
managed = true

```
#### 1.1.2 Sincronizar entorno (Entorno virtual)
```python
uv sync
```
#### 1.1.3 Ejecutar en local
```python
source .venv/bin/activate
WELCOME_MSG="saludos terricolas" uv run devops.py
deactivate
```
# 2 Crear Template
## 2.1 Capturar codigo
```python
CODIGO=$(cat index.py)
```
## 2.2 Crea template
```python
cat << EOF > $CF_TEMPLATE
AWSTemplateFormatVersion: '2010-09-09'
Description: Creando API de Python con Lambda.

Resources:
  # -------------------------------------------------------------------
  # 1. IAM ROLE PARA LAMBDA
  # -------------------------------------------------------------------
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: ${ROLE_NAME}
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

  # -------------------------------------------------------------------
  # 2. LAMBDA FUNCTION
  # -------------------------------------------------------------------
  LambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: ${LAMBDA_NAME}
      Runtime: $RUNTIME
      Handler: index.lambda_handler
      MemorySize: ${MEMORY}
      Timeout: ${TIMEOUT}
      Role: !GetAtt LambdaExecutionRole.Arn
      Environment: 
        Variables: 
         WELCOME_MSG : ""
      Code:
        ZipFile: |
$(echo "$CODIGO" | sed 's/^/          /')
EOF
```
# 3 Crear Stack
```bash
aws cloudformation create-stack \
  --stack-name $STACK_NAME \
  --template-body file://$CF_TEMPLATE \
  --capabilities CAPABILITY_NAMED_IAM \
  --region $REGION
```

```bash
aws cloudformation wait stack-create-complete \
  --stack-name $STACK_NAME \
  --region $REGION
if [ $? -eq 0 ]; then
    echo -e "\033[0;32mStack\033[0m \033[1;31m$STACK_NAME\033[0m \033[0;32mdesplegado con éxito.\033[0m"
else 
   echo -e "\033[0;32mStack\033[0m \033[1;31m$STACK_NAME\033[0m \033[0;32mERROR::despliegue.\033[0m"
fi
```
# 4 Crear URL para LAMBDA FUNCTION
```bash
URL=$(aws lambda create-function-url-config \
    --function-name $LAMBDA_NAME \
    --auth-type NONE \
    --query "FunctionUrl" \
    --output text)
```
#### 4.1 Habilitar permisos
```bash
aws lambda add-permission \
    --function-name $LAMBDA_NAME \
    --statement-id PublicURLInvokeURL \
    --action lambda:InvokeFunctionUrl \
    --principal "*" \
    --function-url-auth-type NONE
```

```bash
aws lambda add-permission \
    --function-name $LAMBDA_NAME \
    --statement-id PublicURLInvokeFunction \
    --action lambda:InvokeFunction \
    --principal "*" \
    --invoked-via-function-url
```
# 5 Verificar
#### 5.1 Verificar Stack
```bash
aws cloudformation describe-stacks \
    --stack-name $STACK_NAME
```
#### 5.2 Verificar Role
```bash
aws iam get-role \
    --role-name $ROLE_NAME
```
#### 5.3 Verificar Lambda
```bash
aws lambda get-function \
    --function-name $LAMBDA_NAME
```
#### 5.4 Curl
```
echo $URL
curl -I $URL
```
# 6 Actualizar codigo
```bash
vi index.py
```

```python
def lambda_handler(event, context):
    content="""<h1>Welcome to KKE AWS Labs!</h1>"""
    return {
        "statusCode": 200,
        "headers": { "Content-Type": "text/html" },
        "body": content
    }
```
#### 6.1 Comprimir codigo
```bash
zip codigo.zip index.py
```
#### 6.2 Actualizar codigo
```bash
aws lambda update-function-code \
    --function-name $LAMBDA_NAME \
    --zip-file fileb://codigo.zip
```



