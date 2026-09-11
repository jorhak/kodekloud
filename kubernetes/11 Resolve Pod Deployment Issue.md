```
A junior DevOps team member encountered difficulties deploying a stack on the Kubernetes cluster. The pod fails to start, presenting errors. Let's troubleshoot and rectify the issue promptly.


There is a pod named webserver, and the container within it is named nginx-container, its utilizing the nginx:latest image.

Additionally, there's a sidecar container named sidecar-container using the ubuntu:latest image.

Identify and address the issue to ensure the pod is in the running state and the application is accessible.

Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```

1. Verificar pod
```
kubectl get pod/webserver
```

2. Describir pod
```
kubectl describe pod/webserver
```

```
Warning  Failed     51s (x4 over 2m25s)  kubelet            Failed to pull image "nginx:latests": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:latests": failed to resolve reference "docker.io/library/nginx:latests": docker.io/library/nginx:latests: not found
```
El error se encuentra en la imagen nginx:latests debemos cambiar por nginx:latest

3. Editar pod
```
kubectl edit pod/webserver
```