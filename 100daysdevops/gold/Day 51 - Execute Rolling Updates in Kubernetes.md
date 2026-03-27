```
An application currently running on the Kubernetes cluster employs the nginx web server. The Nautilus application development team has introduced some recent changes that need deployment. They've crafted an image nginx:1.18 with the latest updates.


Execute a rolling update for this application, integrating the nginx:1.18 image. The deployment is named nginx-deployment.

Ensure all pods are operational post-update.

Note: The kubectl utility on jump_host is set up to operate with the Kubernetes cluster
```

## Inspeccion del deployment
Lo primero que vamos hacer es ver los: deployment, replicaset, pod. Y la descripcion de los **pods**.
```
kubectl get deploy,replicaset,pod
kubectl describe pod <NOMBRE DEL POD>
```

Vemos que esta usando la imagen:
```
Image:          nginx:1.16
```

Tambien vamos a conocer el nombre del container:
```
Containers:
  nginx-container:
```

Y debemos reemplazarla por **nginx:1.18**.

## Actualizar imagen
```
kubectl set image deployment.apps/nginx-deployment nginx-container=nginx:1.18
```

- deployment.apps/nginx-deployment: nombre del deployment
- nginx-container: nombre del contenedor

## Verificar
```
kubectl get pod
kubectl describe pod <NOMBRE DEL POD>
```
