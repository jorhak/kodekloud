```
The Nautilus DevOps team continues to explore serverless architecture by setting up another Lambda function. This time, the task must be completed using the AWS Console to familiarize the team with the web interface. The function will return a custom greeting and demonstrate the capabilities of AWS Lambda effectively.

Create Python Script: Create a Python script named lambda_function.py with a function that returns the body Welcome to KKE AWS Labs! and status code 200.

Zip the Python Script: Zip the script into a file named function.zip.

Create Lambda Function: Create a Lambda function named nautilus-lambda-cli using the zipped file and specify Python as the runtime.

IAM Role: Use the IAM role named lambda_execution_role.

Use AWS CLI which is already configured on the aws-client host.
```
# Variables de entorno
```
NAME_SCRIPT=lambda_function.py
BODY="Welcome to KKE AWS Labs!"
STATUS_CODE=200
ZIP_NAME=function.zip
FUNCTION_NAME=nautilus-lambda-cli
RUNTIME=python3.12
ROLE_NAME=lambda_execution_role
```
# 1 Crear codigo python
```
vi $NAME_SCRIPT
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
zip $ZIP_NAME $NAME_SCRIPT
```
# 2 Verificar ROLE
```
ARN_ROLE=$(aws iam get-role \
    --role-name $ROLE_NAME \
    --query Role.Arn \
    --output text)
```
# 3 Crear LAMBDA FUNCTION
```
aws lambda create-function \
    --function-name $FUNCTION_NAME \
    --runtime $RUNTIME \
    --role $ARN_ROLE \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://$ZIP_NAME
```

```
aws lambda wait function-exists \
    --function-name $FUNCTION_NAME
```
#### Crear URL para LAMBDA FUNCTION
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
# 4 Verificar
Abrimos un navegador y colocamos la URL
```
echo $URL
curl -i $URL
```

Verificamos que la funcion lambda fue creada
```
aws lambda get-function \
    --function-name $FUNCTION_NAME
```