```
There are some applications that need to be deployed on Kubernetes cluster and these apps have some pre-requisites where some configurations need to be changed before deploying the app container. Some of these changes cannot be made inside the images so the DevOps team has come up with a solution to use init containers to perform these tasks during deployment. Below is a sample scenario that the team is going to test first.



Create a Deployment named as ic-deploy-nautilus.


Configure spec as replicas should be 1, labels app should be ic-nautilus, template's metadata lables app should be the same ic-nautilus.


The initContainers should be named as ic-msg-nautilus, use image debian with latest tag and use command '/bin/bash', '-c' and 'echo Init Done - Welcome to xFusionCorp Industries > /ic/ecommerce'. The volume mount should be named as ic-volume-nautilus and mount path should be /ic.


Main container should be named as ic-main-nautilus, use image debian with latest tag and use command '/bin/bash', '-c' and 'while true; do cat /ic/ecommerce; sleep 5; done'. The volume mount should be named as ic-volume-nautilus and mount path should be /ic.


Volume to be named as ic-volume-nautilus and it should be an emptyDir type.


Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.
```

## Create deploy-init.yml
```
vi deploy-init.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-nautilus
spec:
  selector:
    matchLabels:
      app: ic-nautilus
  replicas: 1
  template:
    metadata:
      labels:
        app: ic-nautilus
    spec:
      initContainers:
      - name: ic-msg-nautilus
        image: debian:latest
        command: ["/bin/bash", "-c"] 
        args: ["echo 'Init Done - Welcome to xFusionCorp Industries' > /ic/ecommerce"]
        volumeMounts:
          - name: ic-volume-nautilus
            mountPath: /ic
      containers:
      - name: ic-main-nautilus
        image: debian:latest
        command: ["/bin/bash", "-c"]
        args: ["while true; do cat /ic/ecommerce; sleep 5; done"]
        volumeMounts:
          - name: ic-volume-nautilus
            mountPath: /ic
      volumes:
        - name: ic-volume-nautilus
          emptyDir: {}
```

## Inicializar objetos
```
kubectl apply -f deploy-init.yml
```

## Ver los logs
```
kubectl get pod
kubectl logs <nombre_del_pod> -c ic-main-nautilus
```
