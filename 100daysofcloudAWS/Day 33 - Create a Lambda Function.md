```
The Nautilus DevOps team is embracing serverless architecture by integrating AWS Lambda into their operational tasks. They have decided to deploy a simple Lambda function that will return a custom greeting to demonstrate serverless capabilities effectively. This function is crucial for showcasing rapid deployment and easy scalability features of AWS Lambda to the team.

Create Lambda Function: Create a Lambda function named devops-lambda.

Runtime: Use the Runtime Python.

Deploy: The function should print the body Welcome to KKE AWS Labs!.

Status Code: Ensure the status code is 200.

IAM Role: Create and use the IAM role named lambda_execution_role.

Use the AWS Console to complete this task.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL https://534953745331.signin.aws.amazon.com/console?region=us-east-1
Username kk_labs_user
Password contra
Start Time Sat Jul 18 12:09:42 UTC 2026
End Time Sat Jul 18 13:09:42 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:

toggle button
```
# Variables de entorno
```
FUNCTION_NAME=xfusion-lambda
RUNTIME=python3.12
IAM_ROLE_NAME=lambda_execution_role
REGION=us-east-1
```
# 1 Crear IAM ROLE
#### Crear permisos para AWS LAMBDA politica de confianza
```
vi trust-policy.json
```

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
#### Crear permiso basico
```
vi permissions-policy.json
```

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```
#### Crear ROLE
```
ARN_ROLE=$(aws iam create-role \
    --role-name $IAM_ROLE_NAME \
    --assume-role-policy-document file://trust-policy.json \
    --region $REGION \
    --query "Role.Arn" \
    --output text)
```
#### Adjuntar permisos al ROLE
No tengo permisos para ejecutar este comando
```
aws iam put-role-policy \
    --role-name $IAM_ROLE_NAME \
    --policy-name LambdaBasicPermissions \
    --policy-document file://permissions-policy.json
```
# 2 Crear contenido
```
vi lambda_function.py
```

```
def lambda_handler(event, context):
    html_content = """
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>KKE AWS Labs</title>
        <style>
            body { font-family: sans-serif; background-color: #f4f7f6; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
            .container { text-align: center; background: white; padding: 40px; border-radius: 10px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
            h1 { color: #232f3e; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>Welcome to KKE AWS Labs!</h1>
        </div>
    </body>
    </html>
    """
    return {
        "statusCode": 200,
        "headers": { "Content-Type": "text/html" },
        "body": html_content
    }
```
#### Comprimir fichero
```
zip function.zip lambda_function.py
```
# 3 Crear Function
```
aws lambda create-function \
    --function-name $FUNCTION_NAME \
    --runtime $RUNTIME \
    --role $ARN_ROLE \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip \
    --region $REGION
```

```
aws lambda wait function-exists \
    --function-name $FUNCTION_NAME
```
#### Crear URL para la Function
```
URL=$(aws lambda create-function-url-config \
    --function-name $FUNCTION_NAME \
    --auth-type NONE \
    --query "FunctionUrl" \
    --output text)
```
#### Habilitar permisos
```
aws lambda add-permission \
    --function-name $FUNCTION_NAME \
    --statement-id PublicURLInvokeURL \
    --action lambda:InvokeFunctionUrl \
    --principal "*" \
    --function-url-auth-type NONE
```

```
aws lambda add-permission \
    --function-name $FUNCTION_NAME \
    --statement-id PublicURLInvokeFunction \
    --action lambda:InvokeFunction \
    --principal "*" \
    --invoked-via-function-url
```
# Verificar
```
aws lambda get-function \
    --function-name $FUNCTION_NAME
```

Nos abrimos un navegador y colocamos la URL o desde la terminal
```
curl -i $URL
```