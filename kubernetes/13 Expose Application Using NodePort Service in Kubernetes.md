```
The Nautilus DevOps team has already deployed a ReplicaSet to host an application that requires a highly available infrastructure. Your task is to expose the application running in the existing ReplicaSet by creating a Kubernetes NodePort Service.

Follow the specifications below to create the Service and ensure the application pods are accessible:


A ReplicaSet named nginx-replicaset is already running in the cluster.

The pods managed by the ReplicaSet use the following labels:
Assign labels app as nginx_app, and type as front-end.

Create a NodePort Service named nginx-service to expose the application.

Set the NodePort to 30080.

Expose port 80 of the application.

Note: Do not delete or modify the configuration of the deployed ReplicaSet application.
```

1. Verificar los **pods,deploy,replicaset,service**:
```
kubectl get pod,deploy,replicaset,service
```

2. Describir **replicaset**:
```
kubectl describe rs/nginx-replicaset
```

3. Crear **service**:
```
vi service.yaml
```

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx_app
    type: front-end
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

4. Aplicar **servicio**:
```
kubectl apply -f service.yaml
```

5. Verificar **service**:
```
kubectl get svc/nginx-service
```

6. Verificar que el **service** se asocio en los **pods**
```
kubectl describe svc/nginx-service
kubectl get endpoints nginx-service
```

7. Port-Forward
```
kubectl port-forward svc/nginx-service --address=0.0.0.0 8090:80
kubectl port-forward svc/nginx-service 8090:80
```

8. Verificar
```
curl localhost:8090
```