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