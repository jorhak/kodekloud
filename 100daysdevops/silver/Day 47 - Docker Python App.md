```
A python app needed to be Dockerized, and then it needs to be deployed on `App Server 3`. We have already copied a `requirements.txt` file (having the app dependencies) under `/python_app/src/` directory on `App Server 3`. Further complete this task as per details mentioned below:  
  

  

1. Create a `Dockerfile` under `/python_app` directory:
    
    - Use any `python` image as the base image.
    - Install the dependencies using `requirements.txt` file.
    - Expose the port `5002`.
    - Run the `server.py` script using `CMD`.  
          
        
2. Build an image named `nautilus/python-app` using this Dockerfile.  
      
    
3. Once image is built, create a container named `pythonapp_nautilus`:
    
    - Map port `5002` of the container to the host port `8096`.  
          
        
4. Once deployed, you can test the app using `curl` command on `App Server 3`.  
      
    

```sh
curl http://localhost:8096/
```

## Ingresar al servidor
```
ssh banner@172.16.238.12
```

## Ir al directorio
```
cd /python_app
```

## Dockerfile
```
sudo vi Dockerfile
```

```
FROM python:3.10
WORKDIR /app
COPY . .
RUN cd src && \
    pip install --no-cache-dir -r requirements.txt
EXPOSE 5002
CMD ["python", "./src/server.py"]
```

## Construir imagen
```
docker buildx build --platform linux/amd64 -t nautilus/python-app .
```

## Construir container
```
docker run -d -p 8096:5002 --name pythonapp_nautilus nautilus/python-app
```

## Verificar 
```
curl http://localhost:8096
```
