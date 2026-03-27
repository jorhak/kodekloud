```
Earlier today, the Nautilus DevOps team deployed a new release for an application. However, a customer has reported a bug related to this recent release. Consequently, the team aims to revert to the previous version.

  

There exists a deployment named `nginx-deployment`; initiate a rollback to the previous revision.

`Note:` The `kubectl` utility on `jump_host` is configured to interact with the Kubernetes cluster.
```

## Inspecionar
```
kubectl get deploy,rs,pod
kubectl describe pod <NOMBRE DEL POD>
```

## Estado del RollingUpdate
```
kubectl rollout status deploy nginx-deployment
kubectl rollout history deploy nginx-deployment
```

## RollBack
```
kubectl rollout undo deploy nginx-deployment --to-revision=1
```

De esta forma hemos volvido a una version anterior.
