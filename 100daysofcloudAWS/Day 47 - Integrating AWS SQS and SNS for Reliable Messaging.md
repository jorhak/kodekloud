```
The Nautilus DevOps team needs to implement priority queuing using Amazon SQS and SNS. The goal is to create a system where messages with different priorities are handled accordingly. You are required to use AWS CloudFormation to deploy the necessary resources in your AWS account. The CloudFormation template should be created on the AWS client host at /root/nautilus-priority-stack.yml, the stack name must be nautilus-priority-stack and it should create the following resources:

Two SQS queues named nautilus-High-Priority-Queue and nautilus-Low-Priority-Queue.
An SNS topic named nautilus-Priority-Queues-Topic.
A Lambda function named nautilus-priorities-queue-function that will consume messages from the SQS queues. The Lambda function code is provided in /root/index.py on the AWS client host. Set the function memory to 128 MB and the timeout to 10 seconds.
An IAM role named lambda_execution_role that provides the necessary permissions for the Lambda function to interact with SQS and SNS.
Once the stack is deployed, to test the same you can publish messages to the SNS topic, invoke the Lambda function and observe the order in which they are processed by the Lambda function. The high-priority message must be processed first.

topicarn=$(aws sns list-topics --query "Topics[?contains(TopicArn, 'nautilus-Priority-Queues-Topic')].TopicArn" --output text)

aws sns publish --topic-arn $topicarn --message 'High Priority message 1' --message-attributes '{"priority" : { "DataType":"String", "StringValue":"high"}}'

aws sns publish --topic-arn $topicarn --message 'High Priority message 2' --message-attributes '{"priority" : { "DataType":"String", "StringValue":"high"}}'

aws sns publish --topic-arn $topicarn --message 'Low Priority message 1' --message-attributes '{"priority" : { "DataType":"String", "StringValue":"low"}}'

aws sns publish --topic-arn $topicarn --message 'Low Priority message 2' --message-attributes '{"priority" : { "DataType":"String", "StringValue":"low"}}'


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://784288272310.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Mon Sep 07 13:30:57 UTC 2026
End Time	Mon Sep 07 14:30:57 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```bash
PREFIX=devops
CF_TEMPLATE=/root/$PREFIX-priority-stack.yml
STACK_NAME=$PREFIX-priority-stack
SQS_HIGH_NAME=$PREFIX-High-Priority-Queue
SQS_LOW_NAME=$PREFIX-Low-Priority-Queue
SNS_TOPIC_NAME=$PREFIX-Priority-Queues-Topic
LAMBDA_NAME=$PREFIX-priorities-queue-function
LAMBDA_FILE=/root/index.py
ROLE_NAME=lambda_execution_role
REGION=us-east-1
```
# 1. Inspeccionar $LAMBDA_NAME
Vamos a realizar unos cambios pequenos para ver los logs, vamos agregar tres **print**.
```bash
vi $LAMBDA_FILE
```

```python
import boto3
import os
sqs = boto3.client('sqs')
def delete_message(queue_url, receipt_handle, message):
    response = sqs.delete_message(QueueUrl=queue_url, ReceiptHandle=receipt_handle)
    return "Message " + "'" + message + "'" + " deleted"
    
def poll_messages(queue_url):
    QueueUrl=queue_url
    response = sqs.receive_message(
        QueueUrl=QueueUrl,
        AttributeNames=[],
        MaxNumberOfMessages=1,
        MessageAttributeNames=['All'],
        WaitTimeSeconds=3
    )
    if "Messages" in response:
        receipt_handle=response['Messages'][0]['ReceiptHandle']
        message = response['Messages'][0]['Body']
        delete_response = delete_message(QueueUrl,receipt_handle,message)
        return delete_response
    else:
        return "No more messages to poll"
def lambda_handler(event, context):
    response = poll_messages(os.environ['high_priority_queue'])
    if response == "No more messages to poll":
        response = poll_messages(os.environ['low_priority_queue'])
    return response
```
# 2. Crear Template
## 2.1 Obtener codigo para insertarlo en el template
```bash
LAMBDA_CODE=$(cat $LAMBDA_FILE)
```
## 2.2 Generar fichero 
```bash
cat << EOF > $CF_TEMPLATE
AWSTemplateFormatVersion: '2010-09-09'
Description: Prioridad de colas con SQS, SNS, and Lambda.

Resources:
  # -------------------------------------------------------------------
  # 1. SQS Colas
  # -------------------------------------------------------------------
  HighPriorityQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 180
      QueueName: ${SQS_HIGH_NAME}

  LowPriorityQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 180
      QueueName: ${SQS_LOW_NAME}

  # -------------------------------------------------------------------
  # 2. SNS TOPIC y SUBSCRIPCION CON POLITICAS DE FILTRO
  # -------------------------------------------------------------------
  PriorityTopic:
    Type: AWS::SNS::Topic
    Properties:
      TopicName: ${SNS_TOPIC_NAME}

  HighPrioritySubscription:
    Type: AWS::SNS::Subscription
    Properties:
      TopicArn: !Ref PriorityTopic
      Protocol: sqs
      Endpoint: !GetAtt HighPriorityQueue.Arn
      RawMessageDelivery: true
      FilterPolicy:
        priority:
          - high

  LowPrioritySubscription:
    Type: AWS::SNS::Subscription
    Properties:
      TopicArn: !Ref PriorityTopic
      Protocol: sqs
      Endpoint: !GetAtt LowPriorityQueue.Arn
      RawMessageDelivery: true
      FilterPolicy:
        priority:
          - low

  # Politica para SNS para enviear mensajes a la cola de alta prioridad
  HighPriorityQueuePolicy:
    Type: AWS::SQS::QueuePolicy
    Properties:
      Queues:
        - !Ref HighPriorityQueue
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: sns.amazonaws.com
            Action: sqs:SendMessage
            Resource: !GetAtt HighPriorityQueue.Arn
            Condition:
              ArnEquals:
                aws:SourceArn: !Ref PriorityTopic

  # Politica para SNS para enviear mensajes a la cola de baja prioridad
  LowPriorityQueuePolicy:
    Type: AWS::SQS::QueuePolicy
    Properties:
      Queues:
        - !Ref LowPriorityQueue
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: sns.amazonaws.com
            Action: sqs:SendMessage
            Resource: !GetAtt LowPriorityQueue.Arn
            Condition:
              ArnEquals:
                aws:SourceArn: !Ref PriorityTopic

  # -------------------------------------------------------------------
  # 3. IAM ROLE PARA LAMBDA
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
        - arn:aws:iam::aws:policy/AmazonSQSFullAccess
        - arn:aws:iam::aws:policy/AmazonSNSFullAccess

  # -------------------------------------------------------------------
  # 4. LAMBDA FUNCTION
  # -------------------------------------------------------------------
  PriorityLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: ${LAMBDA_NAME}
      Runtime: python3.12
      Handler: index.lambda_handler
      MemorySize: 128
      Timeout: 10
      Role: !GetAtt LambdaExecutionRole.Arn
      Environment: 
        Variables: 
          high_priority_queue: !Ref HighPriorityQueue 
          low_priority_queue: !Ref LowPriorityQueue
      Code:
        ZipFile: |
$(echo "$LAMBDA_CODE" | sed 's/^/          /')

  # -------------------------------------------------------------------
  # 5. SQS EVENT SOURCE MAPPINGS (TRIGGERS)
  # -------------------------------------------------------------------
  HighPriorityEventSourceMapping:
    Type: AWS::Lambda::EventSourceMapping
    Properties:
      BatchSize: 10
      EventSourceArn: !GetAtt HighPriorityQueue.Arn
      FunctionName: !Ref PriorityLambdaFunction
      Enabled: true

  LowPriorityEventSourceMapping:
    Type: AWS::Lambda::EventSourceMapping
    Properties:
      BatchSize: 10
      EventSourceArn: !GetAtt LowPriorityQueue.Arn
      FunctionName: !Ref PriorityLambdaFunction
      Enabled: true

Outputs:
  TopicArn:
    Value: !Ref PriorityTopic
  HighQueueArn:
    Value: !GetAtt HighPriorityQueue.Arn
  LowQueueArn:
    Value: !GetAtt LowPriorityQueue.Arn
EOF
```
# 3. Crear Stack con CloudFormation
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

echo -e "\033[0;32mStack\033[0m \033[1;31m$STACK_NAME\033[0m \033[0;32mdesplegado con éxito.\033[0m"
```
## 3.1 Eliminar Stack
```bash
aws cloudformation delete-stack \
    --stack-name $STACK_NAME \
    --deletion-mode FORCE_DELETE_STACK \
    --region $REGION
```

```bash
aws cloudformation delete-stack \
    --stack-name $STACK_NAME \
    --region $REGION
aws cloudformation wait stack-delete-complete \
    --stack-name $STACK_NAME \
    --region $REGION
```
# 4. Verificar
## 4.1 Enviar mensajes
```bash
topicarn=$(aws sns list-topics --query "Topics[?contains(TopicArn, '$SNS_TOPIC_NAME')].TopicArn" --output text --region $REGION)
```

```bash
aws sns publish \
    --topic-arn $topicarn \
    --message 'High Priority message 1' \
    --message-attributes '{"priority" : { "DataType":"String", "StringValue":"high"}}' \
    --region $REGION
    
aws sns publish \
    --topic-arn $topicarn \
    --message 'High Priority message 2' \
    --message-attributes '{"priority" : { "DataType":"String", "StringValue":"high"}}' \
    --region $REGION
    
aws sns publish \
    --topic-arn $topicarn \
    --message 'Low Priority message 1' \
    --message-attributes '{"priority" : { "DataType":"String", "StringValue":"low"}}' \
    --region $REGION
    
aws sns publish \
    --topic-arn $topicarn \
    --message 'Low Priority message 2' \
    --message-attributes '{"priority" : { "DataType":"String", "StringValue":"low"}}' \
    --region $REGION
```
## 4.2 Ver logs
Abrimos otra terminal y ejecutamos **Variables de entorno** y luego:
```bash
LOG_STREAM=$(aws logs describe-log-streams \
    --log-group-name "/aws/lambda/$LAMBDA_NAME" \
    --order-by LastEventTime \
    --descending \
    --limit 4 \
    --region $REGION \
    --query "logStreams[*].logStreamName" \
    --output text)  

for LOG in $LOG_STREAM 
do
aws logs get-log-events \
    --log-group-name "/aws/lambda/$LAMBDA_NAME" \
    --log-stream-name "$LOG" \
    --region $REGION \
    --output text \
    --query "events[].message"
done
```

## (Opcional) Modificar index.py
```bash
vi $LAMBDA_FILE
```

```python
import boto3
import os

sqs = boto3.client('sqs')

def delete_message(queue_url, receipt_handle, message):
    # Procesa la confirmación de eliminación e imprime el log en CloudWatch
    result = "Message " + "'" + message + "'" + " deleted"
    print(f"Delete Message::Delete::{result}")
    return result

def poll_messages(records):
    if not records:
        return "No more messages to poll"
    
    # Extraer el primer registro capturado por el EventSourceMapping de Lambda
    record = records[0]
    message_body = record.get('body', '')
    receipt_handle = record.get('receiptHandle', '')
    event_source_arn = record.get('eventSourceARN', '')

    # Determinar qué cola originó el mensaje
    if "High-Priority" in event_source_arn:
        queue_url = os.environ.get('high_priority_queue')
    else:
        queue_url = os.environ.get('low_priority_queue')

    # Ejecutar el flujo de borrado con la firma de tu función
    delete_response = delete_message(queue_url, receipt_handle, message_body)
    print(f"Pool Message::INFO::{delete_response}")
    return delete_response

def lambda_handler(event, context):
    records = event.get('Records', [])
    response = poll_messages(records)
    print(f"Lambda Handler::INFO::{response}")
    return response
```
## (Opcional) Actualizar codigo en Lambda
```bash
cd /root
zip -q /tmp/lambda.zip index.py

aws lambda update-function-code \
  --function-name $LAMBDA_NAME \
  --zip-file fileb:///tmp/lambda.zip \
  --region $REGION
```
ERROR QUE ME SALE CON MI IMPLEMENTACION
```
as e:
```
# Version 2
# Variables de entorno
```bash
PREFIX=devops
CF_TEMPLATE=/root/$PREFIX-priority-stack.yml
STACK_NAME=$PREFIX-priority-stack
SQS_HIGH_NAME=$PREFIX-High-Priority-Queue
SQS_LOW_NAME=$PREFIX-Low-Priority-Queue
SNS_TOPIC_NAME=$PREFIX-Priority-Queues-Topic
LAMBDA_NAME=$PREFIX-priorities-queue-function
LAMBDA_FILE=/root/index.py
ROLE_NAME=lambda_execution_role
REGION=us-east-1
```
# 1 Crear template pagina oficial
```bash
cat << EOF > $CF_TEMPLATE
AWSTemplateFormatVersion: '2010-09-09'
Description: SQS priority queues template

Resources:
  SQSHighPriorityQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 180
      QueueName: ${SQS_HIGH_NAME}

  SQSLowPriorityQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 180
      QueueName: ${SQS_LOW_NAME}

  PriorityQueuesTopic:
    Type: AWS::SNS::Topic
    Properties: 
      TopicName: ${SNS_TOPIC_NAME}

  SQSHighQueuePolicy:
    Type: AWS::SQS::QueuePolicy
    Properties:
      Queues:
        - !Ref SQSHighPriorityQueue
      PolicyDocument:
        Id: AllowIncomingMessageFromSNS
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - sqs:SendMessage
            Resource:
              - !GetAtt SQSHighPriorityQueue.Arn
            Condition:
              ArnEquals:
                aws:SourceArn: !Ref PriorityQueuesTopic

  SQSLowQueuePolicy:
    Type: AWS::SQS::QueuePolicy
    Properties:
      Queues:
        - !Ref SQSLowPriorityQueue
      PolicyDocument:
        Id: AllowIncomingMessageFromSNS
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - sqs:SendMessage
            Resource:
              - !GetAtt SQSLowPriorityQueue.Arn
            Condition:
              ArnEquals:
                aws:SourceArn: !Ref PriorityQueuesTopic

  SNSHighSubscription:
    Type: AWS::SNS::Subscription
    Properties:
      TopicArn: !Ref PriorityQueuesTopic
      Endpoint: !GetAtt SQSHighPriorityQueue.Arn
      Protocol: sqs
      RawMessageDelivery: true
      FilterPolicy: {"priority": ["high"]}

  SNSLowSubscription:
    Type: AWS::SNS::Subscription
    Properties:
      TopicArn: !Ref PriorityQueuesTopic
      Endpoint: !GetAtt SQSLowPriorityQueue.Arn
      Protocol: sqs
      RawMessageDelivery: true
      FilterPolicy: {"priority": ["low"]}

  LambdaRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: ${ROLE_NAME}
      AssumeRolePolicyDocument:
        Statement:
          - Action:
              - sts:AssumeRole
            Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
        Version: 2012-10-17
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
        - arn:aws:iam::aws:policy/AmazonSQSFullAccess
        - arn:aws:iam::aws:policy/AmazonSNSFullAccess
      Path: /

  LambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: ${LAMBDA_NAME}
      Description: Priority queue function
      Runtime: python3.9
      Code:
        ZipFile: >
          import boto3 
          import os
          sqs = boto3.client('sqs')
          def delete_message(queue_url, receipt_handle, message):
              response = sqs.delete_message(QueueUrl=queue_url, ReceiptHandle=receipt_handle)
              return "Message " + "'" + message + "'" + " deleted"
              
          def poll_messages(queue_url):
              QueueUrl=queue_url
              response = sqs.receive_message(
                  QueueUrl=QueueUrl,
                  AttributeNames=[],
                  MaxNumberOfMessages=1,
                  MessageAttributeNames=['All'],
                  WaitTimeSeconds=3
              )
              if "Messages" in response:
                  receipt_handle=response['Messages'][0]['ReceiptHandle']
                  message = response['Messages'][0]['Body']
                  delete_response = delete_message(QueueUrl,receipt_handle,message)
                  return delete_response
              else:
                  return "No more messages to poll"

          def lambda_handler(event, context):
              response = poll_messages(os.environ['high_priority_queue'])
              if response == "No more messages to poll":
                  response = poll_messages(os.environ['low_priority_queue'])
              return response

      Handler: index.lambda_handler
      MemorySize: 128
      Timeout: 10
      Role:
        Fn::GetAtt:
          - LambdaRole
          - Arn
      Environment:
        Variables:
          high_priority_queue: !Ref SQSHighPriorityQueue
          low_priority_queue: !Ref SQSLowPriorityQueue

  HighPriorityEventSource:
    Type: AWS::Lambda::EventSourceMapping
    Properties:
      EventSourceArn: !GetAtt SQSHighPriorityQueue.Arn
      FunctionName: !Ref LambdaFunction
      BatchSize: 1
      Enabled: true

Outputs:
  SNSTopicARN:
    Value: !Ref PriorityQueuesTopic
EOF
```

# 2. Crear Stack con CloudFormation
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
# 2.1 Update (SOLO SI HACES CAMBIOS)
```bash
aws cloudformation update-stack \
    --stack-name $STACK_NAME \
    --template-body file://$CF_TEMPLATE \
    --capabilities CAPABILITY_NAMED_IAM \
    --region $REGION
```
## 2.2 Eliminar Stack (SOLO SI ES NECESARIO)
```bash
aws cloudformation delete-stack \
    --stack-name $STACK_NAME \
    --deletion-mode FORCE_DELETE_STACK \
    --region $REGION
```

```bash
aws cloudformation delete-stack \
    --stack-name $STACK_NAME \
    --region $REGION
aws cloudformation wait stack-delete-complete \
    --stack-name $STACK_NAME \
    --region $REGION
```
No puedo eliminar el stack por completo porque no tengo permisos para eliminar:
```bash
Resource handler returned message: "User: arn:aws:iam::459761813056:user/kk_labs_user_397220 is not authorized to perform: lambda:DeleteEventSourceMapping on resource: arn:aws:lambda:us-east-1:459761813056:event-source-mapping:b0888b65-ccde-4b4d-beb4-af932f55b580 because no identity-based policy allows the lambda:DeleteEventSourceMapping action (Service: Lambda, Status Code: 403, Request ID: b210f3dd-11bf-4811-9788-3b11eef83a78) (SDK Attempt Count: 1)" (RequestToken: 50a8f814-c30f-993a-fa54-f32ea22a7710, HandlerErrorCode: GeneralServiceException)
```
# 3. Verificar
## 3.1 Enviar mensajes
```bash
topicarn=$(aws sns list-topics --query "Topics[?contains(TopicArn, '$SNS_TOPIC_NAME')].TopicArn" --output text --region $REGION)
```

```bash
aws sns publish \
    --topic-arn $topicarn \
    --message 'High Priority message 1' \
    --message-attributes '{"priority" : { "DataType":"String", "StringValue":"high"}}' \
    --region $REGION
    
aws sns publish \
    --topic-arn $topicarn \
    --message 'High Priority message 2' \
    --message-attributes '{"priority" : { "DataType":"String", "StringValue":"high"}}' \
    --region $REGION
    
aws sns publish \
    --topic-arn $topicarn \
    --message 'Low Priority message 1' \
    --message-attributes '{"priority" : { "DataType":"String", "StringValue":"low"}}' \
    --region $REGION
    
aws sns publish \
    --topic-arn $topicarn \
    --message 'Low Priority message 2' \
    --message-attributes '{"priority" : { "DataType":"String", "StringValue":"low"}}' \
    --region $REGION
```
## 3.2 Ver logs
Abrimos otra terminal y ejecutamos **Variables de entorno** y luego:
```bash
LOG_STREAM=$(aws logs describe-log-streams \
    --log-group-name "/aws/lambda/$LAMBDA_NAME" \
    --order-by LastEventTime \
    --descending \
    --limit 2 \
    --region $REGION \
    --query "logStreams[*].logStreamName" \
    --output text)  

for LOG in $LOG_STREAM 
do
aws logs get-log-events \
    --log-group-name "/aws/lambda/$LAMBDA_NAME" \
    --log-stream-name "$LOG" \
    --region $REGION \
    --output text \
    --query "events[].message"
    echo "====================="
done
```

```bash
INIT_START Runtime Version: python:3.9.v133     Runtime Version ARN: arn:aws:lambda:us-east-1::runtime:b46f7bc0f3da8071d1b824471f2c69c8766b756b827eb0455d2118c622ae7bcf
        [WARNING]       2026-09-11T22:35:55.740Z                LAMBDA_WARNING: Unhandled exception. The most likely cause is an issue in the function code. However, in rare cases, a Lambda runtime update can cause unexpected function behavior. For functions using managed runtimes, runtime updates can be triggered by a function change, or can be applied automatically. To determine if the runtime has been updated, check the runtime version in the INIT_START log entry. If this error correlates with a change in the runtime version, you may be able to mitigate this error by temporarily rolling back to the previous runtime version. For more information, see https://docs.aws.amazon.com/lambda/latest/dg/runtimes-update.html
        [ERROR] Runtime.UserCodeSyntaxError: Syntax error in module 'index': invalid syntax (index.py, line 1)
Traceback (most recent call last):
  File "/var/task/index.py" Line 1
    import boto3  import os sqs = boto3.client('sqs') def delete_message(queue_url, receipt_handle, message):       INIT_REPORT Init Duration: 118.06 msPhase: init     Status: error   Error Type: Runtime.UserCodeSyntaxError
        [WARNING]       2026-09-11T22:35:55.847Z                LAMBDA_WARNING: Unhandled exception. The most likely cause is an issue in the function code. However, in rare cases, a Lambda runtime update can cause unexpected function behavior. For functions using managed runtimes, runtime updates can be triggered by a function change, or can be applied automatically. To determine if the runtime has been updated, check the runtime version in the INIT_START log entry. If this error correlates with a change in the runtime version, you may be able to mitigate this error by temporarily rolling back to the previous runtime version. For more information, see https://docs.aws.amazon.com/lambda/latest/dg/runtimes-update.html
        [ERROR] Runtime.UserCodeSyntaxError: Syntax error in module 'index': invalid syntax (index.py, line 1)
Traceback (most recent call last):
  File "/var/task/index.py" Line 1
    import boto3  import os sqs = boto3.client('sqs') def delete_message(queue_url, receipt_handle, message):       INIT_REPORT Init Duration: 80.25 msPhase: invoke    Status: error   Error Type: Runtime.UserCodeSyntaxError
        START RequestId: e35a64bc-318d-588d-bfda-07a2b5048fd6 Version: $LATEST
        END RequestId: e35a64bc-318d-588d-bfda-07a2b5048fd6
        REPORT RequestId: e35a64bc-318d-588d-bfda-07a2b5048fd6  Duration: 95.79 ms  Billed Duration: 96 ms  Memory Size: 128 MB     Max Memory Used: 40 MB      Status: error   Error Type: Runtime.UserCodeSyntaxError
        [WARNING]       2026-09-11T22:35:56.432Z                LAMBDA_WARNING: Unhandled exception. The most likely cause is an issue in the function code. However, in rare cases, a Lambda runtime update can cause unexpected function behavior. For functions using managed runtimes, runtime updates can be triggered by a function change, or can be applied automatically. To determine if the runtime has been updated, check the runtime version in the INIT_START log entry. If this error correlates with a change in the runtime version, you may be able to mitigate this error by temporarily rolling back to the previous runtime version. For more information, see https://docs.aws.amazon.com/lambda/latest/dg/runtimes-update.html
        [ERROR] Runtime.UserCodeSyntaxError: Syntax error in module 'index': invalid syntax (index.py, line 1)
Traceback (most recent call last):
  File "/var/task/index.py" Line 1
    import boto3  import os sqs = boto3.client('sqs') def delete_message(queue_url, receipt_handle, message):       INIT_REPORT Init Duration: 78.48 msPhase: invoke    Status: error   Error Type: Runtime.UserCodeSyntaxError
        START RequestId: aa52cf36-1186-51ed-bb60-ac28950f75fb Version: $LATEST
        END RequestId: aa52cf36-1186-51ed-bb60-ac28950f75fb
        REPORT RequestId: aa52cf36-1186-51ed-bb60-ac28950f75fb  Duration: 93.52 ms  Billed Duration: 94 ms  Memory Size: 128 MB     Max Memory Used: 40 MB      Status: error   Error Type: Runtime.UserCodeSyntaxError
```
# Version 3
```bash
https://github.com/mirakib/100-days-of-cloud-AWS
```
Al parecer no habia que colocar disparadores (triggers).