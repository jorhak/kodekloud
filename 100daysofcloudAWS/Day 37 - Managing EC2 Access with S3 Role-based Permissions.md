```
The Nautilus DevOps team needs to set up an application on an EC2 instance to interact with an S3 bucket for storing and retrieving data. To achieve this, the team must create a private S3 bucket, set appropriate IAM policies and roles, and test the application functionality.

Task:

1) EC2 Instance Setup:
An instance named devops-ec2 already exists.
The instance requires access to an S3 bucket.

2) Setup SSH Keys:
Create new SSH key pair (id_rsa and id_rsa.pub) on the aws-client host and add the public key to the root user's authorized keys on the EC2 instance.

3) Create a Private S3 Bucket:
Name the bucket devops-s3-254222094606.
Ensure the bucket is private.

4) Create an IAM Policy and Role:
Create an IAM policy allowing s3:PutObject, s3:ListBucket and s3:GetObject access to devops-s3-254222094606.

Create an IAM role named devops-role.

Attach the policy to the IAM role.

Attach this role to the devops-ec2 instance.
 

5) Test the Access:
SSH into the EC2 instance and try to upload a file to devops-s3-254222094606 bucket using following command:
aws s3 cp <your-file> s3://devops-s3-254222094606/

Now run following command to list the upload file:
aws s3 ls s3://devops-s3-254222094606/

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL https://254222094606.signin.aws.amazon.com/console?region=us-east-1

Username kk_labs_user
Password contra
Start Time Thu Aug 06 02:11:17 UTC 2026

End Time Thu Aug 06 03:11:17 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:

toggle button
```
# Variables de entorno
```
PREFIX=datacenter
INSTANCE_NAME=$PREFIX-ec2
S3_NAME=$PREFIX-s3-045946725843
POLICY_NAME=$PREFIX-policy
ROLE_NAME=$PREFIX-role
PROFILE_NAME=$PREFIX-profile
REGION=us-east-1
```
# Obtener Instance ID
```
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].InstanceId" \
    --output text)
```
# 1 Ver instancia (Opcional)
```
aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME
```
## 1.1 Ver imagen de la instancia (Opcional)
Del comando anterior obtenemos el ID de la imagen
```
aws ec2 describe-images \
    --image-ids ami-0a0e5d9c7acc336f1 \
    --region $REGION
```
# 2 Configurar SSH
## 2.1 Crear par de llaves
```
ssh-keygen -t rsa -b 4096
```
## 2.2 Obtener IP publica y usuario
```
IP_PUBLIC=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME \
    --query "Reservations[].Instances[].PublicIpAddress" \
    --output text)
USER=ubuntu
```
## 2.3 Copiar llave publica
Obtenemos la llave publica y la copiamos
```
cat ~/.ssh/id_rsa.pub
```
## 2.4 Ingresar a la consola de AWS
EC2>Instances>$INSTANCE_NAME damos click en Connect seleccionamos la pestana In web browser y elegimos EC2 Instance Connect, en Settings lo dejamos en IPv4 finalmente damos click en Connect.

Se nos va abrir una terminal y lo que debemos hacer es copiar la llave publica en authorized_keys de ambos usuarios: ubuntu y root.
```
vi ~/.ssh/authorized_keys
sudo su
vi ~/.ssh/authorized_keys
```
## 2.5 Verificar que tiene instalado aws-cli
```
exit
aws --version
```
# 3 Crear Bucket
Volvemos a la terminal de KodeKloud
```
aws s3api create-bucket \
    --acl private \
    --bucket $S3_NAME \
    --region $REGION
```
Habilitamos los permisos del Bucket
```
aws s3api put-public-access-block \
    --bucket $S3_NAME \
    --public-access-block-configuration "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
```
# 4 Crear politica
```
cat <<EOF > s3_policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListBucketPermission",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::$S3_NAME"
        },
        {
            "Sid": "ObjectPermissions",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::$S3_NAME/*"
        }
    ]
}
EOF
```

```
POLICY_ARN=$(aws iam create-policy \
    --policy-name $POLICY_NAME \
    --policy-document file://s3_policy.json \
    --query 'Policy.Arn' \
    --output text)
```
# 5 Crear relacion de confianza para que EC2 asuma el Role
```
cat <<EOF > ec2_trust_policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

```
aws iam create-role \
    --role-name $ROLE_NAME \
    --assume-role-policy-document file://ec2_trust_policy.json
```
# 6 Asignar Politica a Role
Con la politica y el rol creado en el paso 4 y 5 respectivamente lo que hace es adjuntar la politica al rol.
```
aws iam attach-role-policy \
    --role-name $ROLE_NAME \
    --policy-arn $POLICY_ARN
```
# 7 Crear Instance Profile
```
aws iam create-instance-profile \
    --instance-profile-name $PROFILE_NAME
```
## 7.1 Anadir el rol
```
aws iam add-role-to-instance-profile \
    --instance-profile-name $PROFILE_NAME \
    --role-name $ROLE_NAME

sleep 10
```
## 7.2 Asociar Instance Profile a EC2
```
aws ec2 associate-iam-instance-profile \
    --instance-id $INSTANCE_ID \
    --iam-instance-profile Name=$PROFILE_NAME
```
# 8 Ingresar al servidor
```
ssh $USER@$IP_PUBLIC
```
## 8.1 Crear un fichero
```
echo 'Este es un fichero de prueba que se sube desde EC2 a S3!!!!' > fichero.txt
```
## 8.2 Subir fichero a S3
Actualizamos las variables de entorno
```
PREFIX=datacenter
S3_NAME=$PREFIX-s3-045946725843
aws s3 cp fichero.txt s3://$S3_NAME/
```
## 8.3 Verificar que se subio el fichero
```
aws s3 ls s3://$S3_NAME
```
**Nos salimos del servidor y deshabilitamos permisos para que sea privado.**
```
aws s3api put-public-access-block \
    --bucket $S3_NAME \
    --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```