```
The Nautilus DevOps team has been tasked with setting up a containerized application. They need to create a Azure Container Registry (ACR) to store their Docker images. Once the repository is created, they will build a Docker image from a Dockerfile located on the azure-client host and push this image to the ACR repository. This process is essential for maintaining and deploying containerized applications in a streamlined manner.

1) Create a ACR repository named datacenteracr32349 under East US.

2) Pricing plan must be Basic.

3) Dockerfile already exists under /root/pyapp directory on azure-client host.

4) Build a Docker image using this Dockerfile and push the same to the newly created ACR repo. The image tag must be latest i.e datacenteracr32349:latest.


Use below given Azure Credentials: (You can run the showcreds command on azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra
Start Time	Thu Mar 26 13:00:59 UTC 2026
End Time	Thu Mar 26 14:00:59 UTC 2026

Notes:

Create the resources only in East US region.

To display or hide the terminal of the Azure client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
ACR_NAME=datacenteracr32349
IMAGE_NAME=datacenteracr32349
TAG=latest
SKU=BASIC
LOCATION=eastus
```

1. Obtener _name_ de **Resource Groups**:
```
RG_NAME=$(az group list \
   --query [0].name \
   --output tsv)
```

2. Crear **Azure Container Registry (ACR)**
```
az acr create \
   -n $ACR_NAME \
   -g $RG_NAME \
   --sku $SKU \
   --location $LOCATION
```

3. Obtener _loginServer_ del **ACR** que acabamos de crear
```
LOGIN_SERVER=$(az acr show \
   -n $ACR_NAME \
   -g $RG_NAME \
   --query loginServer \
   --output tsv)
```

4. Crear imagen
```
docker buildx build --no-cache --platform=linux -t $IMAGE_NAME:$TAG .
```

5. Agregar tag
```
docker tag $IMAGE_NAME:$TAG $LOGIN_SERVER/$IMAGE_NAME:$TAG
```

6. Login **ACR**
```
az acr login --name $ACR_NAME
```

7. Subir imagen
```
docker push $LOGIN_SERVER/$IMAGE_NAME:$TAG
```

8. Verificar que la imagen se subio al repositorio
```
az acr repository list \
   -n $ACR_NAME \
   --output table
```

Opcional
datacenteracr32349 es de la salida del comando previo
```
az acr repository show --name $ACR_NAME --repository <datacenteracr32349>
```
