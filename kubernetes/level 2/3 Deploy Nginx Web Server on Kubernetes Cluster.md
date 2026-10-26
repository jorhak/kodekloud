```
Some of the Nautilus team developers are developing a static website and they want to deploy it on Kubernetes cluster. They want it to be highly available and scalable. Therefore, based on the requirements, the DevOps team has decided to create a deployment for it with multiple replicas. Below you can find more details about it:

1. Create a deployment using nginx image with latest tag only and remember to 
   mention the tag i.e nginx:latest. Name it as nginx-deployment. The container 
   should be named as nginx-container, also make sure replica counts are 3.

2. Create a NodePort type service named nginx-service. The nodePort should be 
   30011.

Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```
# Crear deploy.yml
```bash
vi deploy.yml
```

```yml
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:   
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30011
      protocol: TCP
```
## Ejecutar
```bash
kubectl apply -f deploy.yml
```
# Verificar
```bash
kubectl get pod
kubectl get deploy
kubectl get svc
```
Para ver la pagina que si se encutra funcionando damos click sobre el boton en la parte superior derecha **APP**.