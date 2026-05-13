```
The Nautilus DevOps team has been tasked with setting up a containerized application. They need to create a private Amazon Elastic Container Registry (ECR) repository to store their Docker images. Once the repository is created, they will build a Docker image from a Dockerfile located on the aws-client host and push this image to the ECR repository. This process is essential for maintaining and deploying containerized applications in a streamlined manner.

Create a private ECR repository named xfusion-ecr. There is a Dockerfile under /root/pyapp directory on aws-client host, build a docker image using this Dockerfile and push the same to the newly created ECR repo, the image tag must be latest.


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://933196964255.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Wed May 13 14:28:30 UTC 2026
End Time	Wed May 13 15:28:30 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
ECR_NAME=devops-ecr
REGION=us-east-1
```
# 1 Crear repositorio
```
REPOSITORY_URI=$(aws ecr create-repository \
    --repository-name $ECR_NAME \
    --tags Key=Name,Value='xfusion-repositorio-privado' \
    --region $REGION \
    --query "repository.repositoryUri" \
    --output text)
```
# 2 Ver contenido
```
cat /root/pyapp/app.py
```

```
print("Hello, World!")
```

```
cat /root/pyapp/Dockerfile
```

```
# Sample Dockerfile
FROM python:3.8-slim
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

```
cat /root/pyapp/requirements.txt
```
# 4 Cambiar registry
```
aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $REPOSITORY_URI
```
# 3 Crear imagen
### Imagen local
```
cd pyapp
```

```
docker buildx build --platform linux/amd64 -t app:latest .
```

### Tag a la imagen
```
docker tag app:latest $REPOSITORY_URI:latest
```

# 5 Subir imagen
```
docker push $REPOSITORY_URI:latest
```