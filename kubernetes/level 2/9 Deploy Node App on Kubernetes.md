```
The Nautilus development team has completed development of one of the node applications, which they are planning to deploy on a Kubernetes cluster. They recently had a meeting with the DevOps team to share their requirements. Based on that, the DevOps team has listed out the exact requirements to deploy the app. Find below more details:


1. Create a deployment using kodekloud/centos-ssh-enabled:node image, replica 
   count must be 2.

2. Create a service to expose this app, the service type must be NodePort, 
   targetPort must be 8080 and nodePort should be 30012.

3. Make sure all the pods are in Running state after the deployment.

4. You can check the application by clicking on NodeApp button on top bar.


You can use any labels as per your choice.


Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```
# Crear deploy
```bash
vi deploy.yml
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-deployment-nautilus
spec:
  replicas: 2
  selector:
    matchLabels:
      app: node
  template:
    metadata:
      labels:
        app: node
    spec:
      containers:
      - name: node-container-nautilus
        image: kodekloud/centos-ssh-enabled:node
        ports:
        - containerPort: 8080
```
# Crear service
```bash
vi service.yml
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: node-service-nautilus
spec:
  type: NodePort
  selector:
    app: node
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30012
    protocol: TCP
```
# Ejecutar
```bash
kubectl apply -f .
```
# Verificar
```bash
kubectl get all
```
Finalmente damos click en el boton **NodeApp**.
