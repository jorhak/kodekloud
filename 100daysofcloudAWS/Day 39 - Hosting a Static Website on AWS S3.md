```
The Nautilus DevOps team has been tasked with creating an internal information portal for public access. As part of this project, they need to host a static website on AWS using an S3 bucket. The S3 bucket must be configured for public access to allow external users to access the static website directly via the S3 website URL.

Task Requirements:

Create an S3 bucket named nautilus-web-2946431746.
Configure the S3 bucket for static website hosting with index.html as the index document.
Allow public access to the bucket so that the website is publicly accessible.
Upload the index.html file from the /root/ directory of the AWS client host to the S3 bucket.
Verify that the website is accessible directly through the S3 website URL.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://467239208986.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Thu Aug 13 13:06:50 UTC 2026
End Time	Thu Aug 13 14:06:50 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
PREFIX=nautilus
SUFIX=204526302
S3_NAME=$PREFIX-web-$SUFIX
REGION=us-east-1
```
# 1 Crear Bucket S3 publico
## 1.1 Crear Bucket S3
```
aws s3api create-bucket \
    --bucket $S3_NAME \
    --region $REGION
```
## 1.2 Habilitar acceso publico
```
aws s3api put-public-access-block \
    --bucket $S3_NAME \
    --public-access-block-configuration "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
```
## 1.3 Configurar S3 para que funcione como web estatica
```
aws s3api put-bucket-website \
    --bucket $S3_NAME \
    --website-configuration '{ 
      "ErrorDocument": { 
        "Key": "err.html" 
      }, 
      "IndexDocument": { 
        "Suffix": "index.html" 
      } 
    }'
```
## 1.4 Politica de acceso publico a los ficheros
```
aws s3api put-bucket-policy \
  --bucket $S3_NAME \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::'"$S3_NAME"'/*"
        }
    ]
}' \
  --region $REGION
```
# 2 Subir fichero
```
aws s3 cp index.html s3://$S3_NAME/index.html \
    --content-type "text/html" \
    --region $REGION
```
Obtener tener URL
```
echo "http://$S3_NAME.s3-website-$REGION.amazonaws.com"
```
Probar con curl
```
curl http://$S3_NAME.s3-website-$REGION.amazonaws.com
```
# OPCIONAL
### Subir una imagen
```
curl -i -o image.png <URL de la imagen>
```

```
aws s3 cp image.png s3://$S3_NAME/image/image.png
```

```
echo "http://$S3_NAME.s3-website-$REGION.amazonaws.com/image/image.png"
```
### Subir fichero
```
echo "fichero de prueba" > prueba
```

```
aws s3 cp image.png s3://$S3_NAME/texto/prueba
```

```
echo "http://$S3_NAME.s3-website-$REGION.amazonaws.com/texto/prueba"
```

```
curl -i http://$S3_NAME.s3-website-$REGION.amazonaws.com/texto/prueba
```