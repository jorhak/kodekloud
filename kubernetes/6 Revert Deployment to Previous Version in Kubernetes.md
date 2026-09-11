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