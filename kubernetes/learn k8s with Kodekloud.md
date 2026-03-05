# 4 Set Resource Limits in Kubernetes Pods
```
The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints. To address this, they plan to set limits on resource utilization. Here are the details:


Create a pod named httpd-pod with a container named httpd-container. Use the httpd image with the latest tag (specify as httpd:latest). Set the following resource limits:

Requests: Memory: 15Mi, CPU: 100m

Limits: Memory: 20Mi, CPU: 100m

Note: The kubectl utility on jump_host is configured to operate with the Kubernetes cluster.
```

1. Crear **Pod**:
```
vi pod.yaml
```

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
```

2. Ejecutar **Pod**:
```
kubectl apply -f pod.yaml
```

3. Verificar **Pod**:
```
kubectl get pod httpd-pod
```

4. Eliminar **Pod**:
```
kubectl delete -f pod.yaml
```

# 5 Execute Rolling Updates in Kubernetes
```
An application currently running on the Kubernetes cluster employs the nginx web server. The Nautilus application development team has introduced some recent changes that need deployment. They've crafted an image nginx:1.18 with the latest updates.


Execute a rolling update for this application, integrating the nginx:1.18 image. The deployment is named nginx-deployment.

Ensure all pods are operational post-update.

Note: The kubectl utility on jump_host is set up to operate with the Kubernetes cluster
```

1. Listar **pod y deploy**:
```
kubectl get pod,deploy
```

2. Describir **deploy**
```
kubectl describe deploy nginx-deployment
```
Podemos ver que la imagen que esta utilizando es _nginx:1.16_

3. Actualizar imagen:
- Obtener el nombre del deploy, nombre del container
```
kubectl get deployment nginx-deployment -o yaml
```

- Actualizar imagen
```
kubectl set image deployments/nginx-deployment nginx-container=nginx:1.18
```

4. Verificar cambios:
```
kubectl rollout status deployment/nginx-deployment
kubectl describe deployment nginx-deployment | grep Image
```

# 6 Revert Deployment to Previous Version in Kubernetes
```
Earlier today, the Nautilus DevOps team deployed a new release for an application. However, a customer has reported a bug related to this recent release. Consequently, the team aims to revert to the previous version.


There exists a deployment named nginx-deployment; initiate a rollback to the previous revision.

Note: The kubectl utility on jump_host is configured to interact with the Kubernetes cluster.
```

1. Describir **deployment**:
```
kubectl describe deployment/nginx-deployment
kubectl get deployment/nginx-deployment -o yaml
```

2. Ver historial 
```
kubectl rollout history deployment/nginx-deployment
```

2. Ver estado
```
kubectl rollout status deployment/nginx-deployment
```

3. Rollback
```
kubectl rollout undo deployment/nginx-deployment
## si queremos especificar
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

# 7 Deploy ReplicaSet in Kubernetes Cluster
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