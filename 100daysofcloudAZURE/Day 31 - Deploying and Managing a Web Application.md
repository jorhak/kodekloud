```
The Nautilus DevOps team is tasked with deploying a Python-based web application on Azure. You need to create a web app using the following specifications:

1) The Web App name should be datacenter-webapp.
2) It should be created in the West US region under the default resource group.
3) The publish option should be set to Code.
4) The Runtime Stack should be Python with Linux as the operating system.
5) Create a new App Service Plan named datacenter-learn-python with the SKU Basic B1.
6) Application Insights should be disabled.
7) Add tags:

Name: WebAppLearning
Environment: Dev
Make sure the web app is in Running state after creation.


Use below given Azure Credentials: (You can run the showcreds command on azure-client host to retrieve these credentials)

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azurefree.onmicrosoft.com
Password	contra
Start Time	Tue May 05 15:28:03 UTC 2026
End Time	Tue May 05 16:28:03 UTC 2026
```

# Variables de entorno
```
APP_SERVICE_NAME=datacenter-learn-python
SKU=B1
TAGS="Name=WebAppLearning Environment=Dev"
           WebAppLearning
APP_NAME=datacenter-webapp
LOCATION=westus
RUNTIME="PYTHON:3.10"
OS_NAME=Linux
```

# 1 Obtener Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# 2 Crear App Service Plan
```
az appservice plan create \
   --name $APP_SERVICE_NAME \
   --resource-group $RG_NAME \
   --sku $SKU \
   --is-linux \
   --location $LOCATION \
```

# 3 Crear WebApp
### (Opcional) Ver los RUNTIME permitidos
```
az webapp list-runtimes --os-type linux
```
### Crear WebApp
```
az webapp create \
   --name $APP_NAME \
   --plan $APP_SERVICE_NAME \
   --resource-group $RG_NAME \
   --runtime $RUNTIME \
   --tags $TAGS
```

Nota: Aqui ocurre un error debido a que ya existe una WebApp con ese nombre por lo que lo cambiamos para poder seguir con el laboratorio. Ejecutar solo si sale error:
```
APP_NAME=datacenter-webapp-pier-8374
```
# 4 Ejecutar App
```
mkdir myProject && cd myProject
```
### Codigo main.py
```
from flask import Flask, jsonify

app = Flask(__name__)

# A simple GET endpoint
@app.route('/api/greet', methods=['GET'])
def greet():
    return jsonify({"message": "Hello from Flask API!"})

if __name__ == '__main__':
    app.run(debug=True)

```

### requirements.txt
```
Flask>=3.0.0
```
# 5 Configurar RunTime
```
az webapp config set \
    --resource-group $RG_NAME \
    --name $APP_NAME \
    --startup-file "gunicorn --bind=0.0.0.0 --timeout 600 main:app"
```

# 6 Empaquetar App
```
sudo apt install zip
```

```
zip -r deploy.zip .
```

# 7 Desplegar
```
az webapp deployment source config-zip \
   --resource-group $RG_NAME \
   --name $APP_NAME \
   --src deploy.zip
```
# 8 Probar en navegador
```
az webapp show --name $APP_NAME --resource-group $RG_NAME | grep $APP_NAME
```

Colocamos en el navegador:
```
https://datacenter-webapp.azurewebsites.net/api/greet
```

ERROR:
```
WebAppLearning tag is missing.
```