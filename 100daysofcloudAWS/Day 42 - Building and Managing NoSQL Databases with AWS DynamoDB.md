```
The Nautilus DevOps team is developing a simple 'To-Do' application using DynamoDB to store and manage tasks efficiently. The team needs to create a DynamoDB table to hold tasks, each identified by a unique task ID. Each task will have a description and a status, which indicates the progress of the task (e.g., 'completed' or 'in-progress').

Your task is to:

Create a DynamoDB table named nautilus-tasks with a primary key called taskId (string).
Insert the following tasks into the table:
Task 1: taskId: '1', description: 'Learn DynamoDB', status: 'completed'
Task 2: taskId: '2', description: 'Build To-Do App', status: 'in-progress'
Verify that Task 1 has a status of 'completed' and Task 2 has a status of 'in-progress'.
Ensure the DynamoDB table is created successfully and that both tasks are inserted correctly with the appropriate statuses.


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://709345921074.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Tue Aug 18 13:06:58 UTC 2026
End Time	Tue Aug 18 14:06:58 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
PREFIX=devops
TABLE_NAME=$PREFIX-tasks
PRIMARY_KEY=taskId
PRIMARY_KEY_TYPE=S
```
# 1 Crear tabla DynamoDB
```
aws dynamodb create-table \
    --table-name $TABLE_NAME \
    --attribute-definitions AttributeName=$PRIMARY_KEY,AttributeType=$PRIMARY_KEY_TYPE \
    --key-schema AttributeName=$PRIMARY_KEY,KeyType=HASH \
    --tags Key=Env,Value=DEV Key=Owner,Value=Jorhak \
    --billing-mode PAY_PER_REQUEST
```

```
aws dynamodb wait table-exists --table-name $TABLE_NAME
```
# 2 Insertar datos
## 2.1 Datos
```
cat<<EOF > item1.json
{
  "taskId": {"S": "1"} , 
  "description": {"S": "Learn DynamoDB"} ,
  "status": {"S": "completed"}
}
EOF
```

```
cat<<EOF > item2.json
{
  "taskId": {"S": "2"}, 
  "description": {"S": "Build To-Do App"},
  "status":{"S": "in-progress"}
}
EOF
```
## 2.2 Insertar datos
```
aws dynamodb put-item \
    --table-name $TABLE_NAME \
    --item file://item1.json \
    --return-consumed-capacity TOTAL \
    --return-item-collection-metrics SIZE
```

```
aws dynamodb put-item \
    --table-name $TABLE_NAME \
    --item file://item2.json \
    --return-consumed-capacity TOTAL \
    --return-item-collection-metrics SIZE
```
# 3 Verificar
```
cat<<EOF >key1.json
{
   "taskId": {"S": "1"}
}
EOF
```

```
cat<<EOF >key2.json
{
   "taskId": {"S": "1"}
}
EOF
```

```
aws dynamodb get-item \
    --table-name $TABLE_NAME \
    --key file://key1.json \
    --return-consumed-capacity TOTAL
```

```
aws dynamodb get-item \
    --table-name $TABLE_NAME \
    --key file://key2.json \
    --return-consumed-capacity TOTAL
```