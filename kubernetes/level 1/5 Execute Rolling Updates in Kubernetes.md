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