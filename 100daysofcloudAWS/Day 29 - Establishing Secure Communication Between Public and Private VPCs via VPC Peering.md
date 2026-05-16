```
The Nautilus DevOps team has been tasked with demonstrating the use of VPC Peering to enable communication between two VPCs. One VPC will be a private VPC that contains a private EC2 instance, while the other will be the default public VPC containing a publicly accessible EC2 instance.

1) There is already an existing **EC2 instance** in the public vpc/subnet:

- Name: `nautilus-public-ec2`

2) There is already an existing **Private VPC**:

- Name: `nautilus-private-vpc`
- CIDR: `10.1.0.0/16`

3) There is already an existing **Subnet** in `nautilus-private-vpc`:

- Name: `nautilus-private-subnet`
- CIDR: `10.1.1.0/24`

4) There is already an existing **EC2 instance** in the private subnet:

- Name: `nautilus-private-ec2`

5) **Create a Peering Connection** between the Default VPC and the Private VPC:

- VPC Peering Connection Name: `nautilus-vpc-peering`

6) **Configure Route Tables** to enable communication between the two VPCs.

- Ensure the private EC2 instance is accessible from the public EC2 instance.

7) **Test the Connection**:

- Add `/root/.ssh/id_rsa.pub` public key to the public EC2 instance's `ec2-user`'s `authorized_keys` to make sure we are able to ssh into this instance from AWS client host. You may also need to update the security group of the private EC2 instance to allow ICMP traffic from the public/default VPC CIDR. This will enable you to ping the private instance from the public instance.
- SSH into the public EC2 instance and ensure that you can ping the private EC2 instance.

  

Use below given **AWS Credentials:** (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

|Console URL|[https://417003076066.signin.aws.amazon.com/console?region=us-east-1]|
|---|---|
|Username|kk_labs_user|
|Password|contra|
|Start Time|Thu May 14 19:28:10 UTC 2026|
|End Time|Thu May 14 20:28:10 UTC 2026|

  
`Notes:`

- Create the resources only in `us-east-1` region.
    
- To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:  
    ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
```
# Variables de entorno
```
INSTANCE_NAME_PUBLIC=nautilus-public-ec2
VPC_NAME_PRIVATE=nautilus-private-vpc
VPC_CIDR_PRIVATE=10.1.0.0/16
SUBNET_NAME_PRIVATE=nautilus-private-subnet
SUBNET_CIDR_PRIVATE=10.1.1.0/24
INSTANCE_NAME_PRIVATE=nautilus-private-ec2
VPC_PEERING_NAME=nautilus-vpc-peering
REGION=us-east-1
```

# Listar VPCs
Vamos a listar las VPCs que estan creadas
```
aws ec2 describe-vpcs \
    --query "Vpcs[*].{CIDR:CidrBlock,ID:VpcId,NOMBRE:Tags[?Key=='Name'].Value | [0]}" \
    --output table
```
#### Vpc default
Vamos a ver cual de las VPCs es la que esta por defecto ya que esta por lo general es la VPC publica.
```
aws ec2 describe-vpcs \
    --filters Name=is-default,Values=true \
    --query "Vpcs[0].{CIDR:CidrBlock,ID:VpcId,NOMBRE:Tags[?Key=='Name'].Value | [0]}" \
    --output table
```
Realizamos una comparacion de las dos salidas para determinar cual es la VPC por defecto.
# 1 Obtener VPC ID de la publica y privada
Tenemos la mayoria de las variables de la VPC privada, sin embargo, no las variables de la VPC publica que van a ser necesarias para esta tarea.
#### VPC ID PUBLIC
```
VPC_ID_PUBLIC=$(aws ec2 describe-vpcs \
   --filters Name=is-default,Values=true \
   --query "Vpcs[0].VpcId" \
   --output text)
```
#### VPC CIDR PUBLIC
```
VPC_CIDR_PUBLIC=$(aws ec2 describe-vpcs \
   --vpc-ids $VPC_ID_PUBLIC \
   --query "Vpcs[0].CidrBlock" \
   --output text)
```
#### VPC ID PRIVATE
```
VPC_ID_PRIVATE=$(aws ec2 describe-vpcs \
   --filters "Name=tag:Name,Values=$VPC_NAME_PRIVATE" \
   --query "Vpcs[0].VpcId" \
   --output text)
```
#  2 Crear Peering Connection
Creamos la conexion entre las dos VPCs.
```
PEERING_ID=$(aws ec2 create-vpc-peering-connection \
    --vpc-id $VPC_ID_PUBLIC \
    --peer-vpc-id $VPC_ID_PRIVATE \
    --tag-specifications "ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=$VPC_PEERING_NAME}]" \
    --query "VpcPeeringConnection.VpcPeeringConnectionId" \
    --output text)
```
#### Aceptar conexion
Con este comando vemos el estado de nuestra conexion.
```
aws ec2 accept-vpc-peering-connection \
    --vpc-peering-connection-id $PEERING_ID
```
Buscamos la propiedad **Status**, el cual tiene varios estados: active, provisioning,...
# 3 Configurar tablas de rutas
#### Ruta de la VPC publica a la VPC privada
Ya creamos la conexion, ahora lo que debemos crear es el camino para que esa conexion conosca por donde de ir desde la publica a la privada:
```
RT_ID_PUBLIC=$(aws ec2 describe-route-tables \
   --filters "Name=vpc-id,Values=$VPC_ID_PUBLIC" \
   --query "RouteTables[0].RouteTableId" \
   --output text)
```

```
aws ec2 create-route \
    --route-table-id $RT_ID_PUBLIC \
    --destination-cidr-block $VPC_CIDR_PRIVATE \
    --vpc-peering-connection-id $PEERING_ID
```

#### Ruta de la VPC privada a la VPC publica
Ya creamos la conexion, ahora lo que debemos crear es el camino para que esa conexion conosca por donde de ir desde la privada a la publica:
```
RT_ID_PRIVATE=$(aws ec2 describe-route-tables \
   --filters "Name=vpc-id,Values=$VPC_ID_PRIVATE" \
   --query "RouteTables[0].RouteTableId" \
   --output text)
```

```
aws ec2 create-route \
    --route-table-id $RT_ID_PRIVATE \
    --destination-cidr-block $VPC_CIDR_PUBLIC \
    --vpc-peering-connection-id $PEERING_ID
```
# 4 Configurar NSG de la instancia privada
Lo que hacemos es que la instancia privada (que esta en la VPC privada) acepte peticiones ICMP (ping), para ver si llegamos a tener conexion desde la VPC publica.
#### Obtener NSG_ID
```
NSG_ID_PRIVATE=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$INSTANCE_NAME_PRIVATE" \
    --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" \
    --output text)
```
#### Permitir ICMP desde la VPC Publica
```
aws ec2 authorize-security-group-ingress \
    --group-id $NSG_ID_PRIVATE \
    --protocol icmp \
    --port -1 \
    --cidr $VPC_CIDR_PUBLIC
```
# 5 Ver el estado de las instancias privada y publica
#### Publica
```
aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME_PUBLIC \
    --query "Reservations[0].Instances[0].{Nombre:Tags[?Key=='Name'].Value | [0],IpPublica:PublicIpAddress,IpPrivada:PrivateIpAddress,Estado:State.Name}" \
    --output table
```
#### Privada
```
aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME_PRIVATE \
    --query "Reservations[0].Instances[0].{Nombre:Tags[?Key=='Name'].Value | [0],IpPublica:PublicIpAddress,IpPrivada:PrivateIpAddress,Estado:State.Name}" \
    --output table
```
# 6 Agregar llave publica en la instancia publica
### Opcion 1
Habilitamos el puerto de la instancia publica.
#### Obtener SECURITY GROUP
```
SECURITY_GROUP_NAME=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME_PUBLIC \
    --query "Reservations[0].Instances[0].NetworkInterfaces[0].Groups[0].GroupName" \
    --output text)
```
#### Habilitar puerto SSH
```
aws ec2 authorize-security-group-ingress \
    --group-name $SECURITY_GROUP_NAME \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

Ingresar desde la Consola Web **EC2>Instances>nautilus-public-ec2>Connect>EC2 Instance Connect** marcamos **Connect using a Public IP** y click en *Connect*.
En el navegador se nos abrira una ventana con la terminal:
```
cd .ssh/
vi authorized_keys
```

Nos vamos a nuestra terminal de nuestro host y copiamos la llave publica:
```
cat /root/.ssh/id_rsa.pub
```

Copiamos el contenido de salida y lo pegamos en la terminal del navegador de la instancia publica, guardamos y cerramos la ventana para ingresar desde nuestro host:

```
IP_PUBLIC=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME_PUBLIC \
    --query "Reservations[0].Instances[0].PublicIpAddress" \
    --output text)
USER=ec2-user
```

```
IP_PRIVATE=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME_PRIVATE \
    --query "Reservations[0].Instances[0].PrivateIpAddress" \
    --output text)
```
El valor de esta variable IP_PRIVATE la vamos a ocupar en el siguiente paso luego de ingresar a la instancia publica para ver la conexion con la instancia privada que se encuentra en la VPC privada.

```
echo $IP_PRIVATE
```

```
ssh $USER@$IP_PUBLIC
```
### Opcion 2
Si tenemos los permisos correspondientes podemos realizar esta accion.
```
PUB_KEY=$(cat /root/.ssh/id_rsa.pub)
```

```
INSTANCE_ID_PUBLIC=$(aws ec2 describe-instances \
    --filters Name=tag:Name,Values=$INSTANCE_NAME_PUBLIC \
    --query "Reservations[0].Instances[0].InstanceId" \
    --output text)
```

```
aws ssm send-command \
    --instance-ids $INSTANCE_ID_PUBLIC \
    --document-name "AWS-RunShellScript" \
    --parameters '{"commands":["echo '"$PUB_KEY"' >> /home/ec2-user/.ssh/authorized_keys"]}'
```
# 7 Verificar
Una ves dentro de la instancia publica que se encuentra la la VPC publica, lo que debemos hacer es validar que tenemos conexion con la instancia privada que esta en la VPC privda.
```
ping $IP_PRIVATE
```