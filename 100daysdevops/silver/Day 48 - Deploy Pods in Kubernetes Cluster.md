```
The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:

  

1. Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag. Ensure to specify the tag as `httpd:latest`.
    
2. Set the `app` label to `httpd_app`, and name the container as `httpd-container`.
    

`Note`: The `kubectl` utility on `jump_host` is configured to operate with the Kubernetes cluster.
```

## Crear file pod.yml
```
vi pod.yml
```

## Crear pod
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
   - name: httpd-container
     image: httpd:latest
```

## Inicializar pod
```
kubectl apply -f pod.yml
```

## Verificar la cracion del POD
```
kubectl get pod
```
