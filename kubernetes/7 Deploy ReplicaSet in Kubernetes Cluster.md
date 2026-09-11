```
The Nautilus DevOps team is gearing up to deploy applications on a Kubernetes cluster for migration purposes. A team member has been tasked with creating a ReplicaSet outlined below:



Create a ReplicaSet using httpd image with latest tag (ensure to specify as httpd:latest) and name it httpd-replicaset.


Apply labels: app as httpd_app, type as front-end.


Name the container httpd-container. Ensure the replica count is 4.


Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```

1. Crear **Replicaset**:
```code
nano httpd-replicaset.yaml
```


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: httpd-replicaset 
  labels:
    app: httpd_app
    type: front-end
spec:
  # numero de replicas
  replicas: 4
  # selector para gestionar los pods para esta ReplicaSet
  selector:
    matchLabels:
      tier: frontend
  # template para crear nuevos pods
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: httpd-container
        image: httpd:latest
        ports:
        - containerPort: 80
```

2. Ejecutar
```
kubectl apply -f httpd-replicaset.yaml
```

3. Verificar
```
kubectl get pod,rs
###Describir replicaset
kubectl describe rs/httpd-replicaset
```