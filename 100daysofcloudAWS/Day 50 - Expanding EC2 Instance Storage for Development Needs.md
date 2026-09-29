```
The Nautilus DevOps Team has recently been informed by the Development Team that their EC2 instance is running out of storage space. This instance, crucial for development activities, is named datacenter-ec2 and currently has an attached volume of 8 GiB. To accommodate the increasing data requirements, the storage needs to be expanded to 12 GiB. This change should ensure that the expanded space is immediately available for use within the instance without disrupting ongoing activities.

Identify Volume: Find the volume attached to the datacenter-ec2 instance.

Expand Volume: Increase the volume size from 8 GiB to 12 GiB.

Reflect Changes: Ensure the root (/) partition within the instance reflects the expanded size from 8 GiB to 12 GiB.

SSH Access: Use the key pair located at /root/datacenter-keypair.pem on the aws-client host to SSH into the EC2 instance.



AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://000866466769.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Sat Sep 19 12:48:47 UTC 2026
End Time	Sat Sep 19 13:48:47 UTC 2026
Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```bash
PREFIX=devops
INSTANCE_NAME=$PREFIX-ec2
SIZE=12
KEY_PATH=/root/$PREFIX-keypair.pem
```
# Obtener ID Image
```bash
ID_IMAGE=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[0].Instances[0].ImageId" \
    --output text)
```
# Que imagen tiene
```bash
aws ec2 describe-images \
    --image-ids $ID_IMAGE
```
# Obtener el ID Storage
```bash
ID_VOLUME=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.VolumeId" \
    --output text)
```
# Obtener ID Instance
```bash
ID_INSTANCE=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[0].Instances[0].InstanceId" \
    --output text)
```
# 1 Detener instancia
```bash
aws ec2 stop-instances \
    --instance-ids $ID_INSTANCE
```

```bash
aws ec2 wait instance-stopped \
    --instance-ids $ID_INSTANCE
```
## 1.1 Estado de la instancia
```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[0].Instances[0].{Estado:State.Name}" \
    --output table
```
# 2 Modificar tamano de volumen
## 2.1 Ver tamano actual
```bash
aws ec2 describe-volumes \
    --volume-ids $ID_VOLUME
```
## 2.2 Modificar tamano
```bash
aws ec2 modify-volume \
    --size $SIZE \
    --volume-id $ID_VOLUME 
```
# 3 Inicializar instancia
```bash
aws ec2 start-instances \
    --instance-ids $ID_INSTANCE
```

```bash
aws ec2 wait instance-running \
    --instance-ids $ID_INSTANCE
```
## 3.1 Estado de la instancia
```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[0].Instances[0].{Estado:State.Name}" \
    --output table
```
# 4 Ingresar al servidor
## Obtener IP publica
```bash
IP_PUBLIC=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
    --query "Reservations[0].Instances[0].PublicIpAddress" \
    --output text)
USER=ec2-user
```

```bash
ssh -i $KEY_PATH $USER@$IP_PUBLIC
```
# 5 Verificar
```bash
sudo fdisk -l
df -h
```
