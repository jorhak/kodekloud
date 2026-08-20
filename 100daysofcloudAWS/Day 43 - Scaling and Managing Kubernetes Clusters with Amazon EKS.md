```
The Nautilus DevOps team has been tasked with preparing the infrastructure for a new Kubernetes-based application that will be deployed using Amazon EKS. The team is in the process of setting up an EKS cluster that meets their internal security and scalability standards. They require that the cluster be provisioned using the latest stable Kubernetes version to take advantage of new features and security improvements.

To minimize external exposure, the EKS cluster endpoint must be kept private. Additionally, the cluster needs to use the default VPC with availability zones a, b, and c to ensure high availability across different physical locations.

Your task is to create an EKS cluster named devops-eks, with Custom configuration, use IAM role for the cluster named eksClusterRole. Additionally, ensure that EKS Auto Mode is disabled and that the cluster endpoint access is set to private.

Finally, verify that the EKS cluster is successfully created with the correct configuration and is ready for workloads.


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://552551974974.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Wed Aug 19 13:58:26 UTC 2026
End Time	Wed Aug 19 14:58:26 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
PREFIX=devops
EKS_NAME=$PREFIX-eks
K8S_VERSION=1.36
VPC_NAME=default
ROLE_NAME=eksClusterRole
REGION=us-east-1
ROLE_NODE_NAME=eksNodeRole
NODE_GROUP_NAME=$PREFIX-nodes
```
# Obtner Subnet's y Security Group de la VPC
```
VPC_ID=$(aws ec2 describe-vpcs \
    --filters Name=is-default,Values=true \
    --region $REGION \
    --query "Vpcs[0].VpcId" \
    --output text)
```

```
aws ec2 describe-subnets \
    --filters Name=vpc-id,Values=$VPC_ID Name=availability-zone,Values=${REGION}a,${REGION}b,${REGION}c \
    --query "Subnets[*].{ID:SubnetId,Zona:AvailabilityZone}" \
    --output table
```
Lo que obtengamos de esta salida la vamos a utilizar en el paso 2.1

```
SG_ID=$(aws ec2 describe-security-groups \
    --filters Name=vpc-id,Values=$VPC_ID \
    --query "SecurityGroups[].GroupId" \
    --output text)
```
# 1 Crear Role
## 1.1 Creamos la politica de confianza para EKS
```
cat << 'EOF' > eks-trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```
## 1.2 Crear Role
```
ROLE_ARN=$(aws iam create-role \
   --role-name $ROLE_NAME \
   --assume-role-policy-document file://eks-trust-policy.json \
   --query "Role.Arn" \
   --output text)
```
## 1.3 Agregar politica de EKS al Role
```
aws iam attach-role-policy \
    --role-name $ROLE_NAME \
    --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```
# 2 Crear cluster
## 2.1 Obtener las subnets especificadas
```
SUBNET_IDS=$(aws ec2 describe-subnets \
    --filters Name=vpc-id,Values=$VPC_ID Name=availability-zone,Values=${REGION}a,${REGION}b,${REGION}c \
    --query "Subnets[*].SubnetId" \
    --output text | tr '\t' ',')
```
## 2.2 Crear cluster EKS
```
aws eks create-cluster \
    --name $EKS_NAME \
    --role-arn $ROLE_ARN \
    --resources-vpc-config subnetIds=$SUBNET_IDS,securityGroupIds=$SG_ID,endpointPublicAccess=false,endpointPrivateAccess=true \
    --control-plane-scaling-config tier=standard \
    --kubernetes-version $K8S_VERSION \
    --access-config bootstrapClusterCreatorAdminPermissions=true,authenticationMode=API_AND_CONFIG_MAP \
    --region $REGION
```

```
aws eks wait cluster-active \
    --name $EKS_NAME
```

# 3 Crear Worker Nodes (OPCIONAL)
Abrimos otra terminal y ejecutamos el paso **Variables de entorno**
## 3.1 Crear politica de confianz
```
cat << 'EOF' > node-trust-policy.json
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
## 3.2 Crear rol
```
NODE_ROLE_ARN=$(aws iam create-role \
    --role-name $ROLE_NODE_NAME \
    --assume-role-policy-document file://node-trust-policy.json \
    --query "Role.Arn" \
    --output text)
```
## 3.3 Agregar politicas requeridas
```
aws iam attach-role-policy --role-name $ROLE_NODE_NAME --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
aws iam attach-role-policy --role-name $ROLE_NODE_NAME --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
aws iam attach-role-policy --role-name $ROLE_NODE_NAME --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
```
Ejecutamos el comando del paso 2.1 para obtener las subnets.
# 4 Crear Node Group (OPCIONAL)
```
aws eks create-nodegroup \
    --cluster-name $EKS_NAME \
    --nodegroup-name $NODE_GROUP_NAME \
    --subnets $(echo $SUBNET_IDS | tr ',' ' ') \
    --node-role $NODE_ROLE_ARN \
    --scaling-config minSize=1,maxSize=2,desiredSize=1 \
    --region $REGION
```
# 5 Verificar
## 5.1 Verificar que el cluster esta creado
```
aws eks describe-cluster \
    --name $EKS_NAME \
    --query "cluster.{Nombre:name,Version:version,Estado:status,Publico:resourcesVpcConfig.endpointPublicAccess,Privado:resourcesVpcConfig.endpointPrivateAccess,Rol:roleArn,Subnets:resourcesVpcConfig.subnetIds[*]}" \
    --output table \
    --region $REGION
```
## 5.2 Verificar que el rol tiene los permisos para la creacion de un cluster
### 5.2.1 Ver el rol
```
aws iam get-role \
    --role-name $ROLE_NAME
```
### 5.2.2 Que politicas tiene el rol
```
aws iam list-attached-role-policies \
    --role-name $ROLE_NAME
```
### 5.2.3 Vamos a realizar el seguimiento de las politicas que tiene nuestro rol de EKS
```
POLICY_ARN=$(aws iam list-attached-role-policies \
    --role-name $ROLE_NAME \
    --query "AttachedPolicies[0].PolicyArn" \
    --output text)

VERSION_ID=$(aws iam get-policy \
    --policy-arn $POLICY_ARN \
    --query "Policy.DefaultVersionId" \
    --output text)

aws iam get-policy-version \
    --policy-arn $POLICY_ARN \
    --version-id $VERSION_ID --query "PolicyVersion.Document"
```
### 5.2.4 Vamos a realizar el seguimiento de las politicas que tiene nuestro rol de Node Group (OPCIONAL)
```
POLICY_NODE_ARN=$(aws iam list-attached-role-policies \
    --role-name $ROLE_NODE_NAME \
    --query "AttachedPolicies[0].PolicyArn" \
    --output text)

VERSION_ID=$(aws iam get-policy \
    --policy-arn $POLICY_NODE_ARN \
    --query "Policy.DefaultVersionId" \
    --output text)

aws iam get-policy-version \
    --policy-arn $POLICY_NODE_ARN \
    --version-id $VERSION_ID --query "PolicyVersion.Document"
```

# 5.2 Habilitar acceso publico temporal (OPCINAL)
```
aws eks update-cluster-config \
  --name $EKS_NAME \
  --region $REGION \
  --resources-vpc-config endpointPublicAccess=true,endpointPrivateAccess=true
```
# 5.3 Instalar kubectl (OPCIONAL)
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl
```

Los lugares donde tenemos (OPCIONAL) es porque no tenemos los premisos correspondientes para realizar esas acciones.