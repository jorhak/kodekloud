```
We are working on an application that will be deployed on multiple containers within a pod on Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario. Below you can find more details about it.  
  

  

1. Create a pod named `volume-share-devops`.  
      
    
2. For the first container, use image `ubuntu` with `latest` tag only and remember to mention the tag i.e `ubuntu:latest`, container should be named as `volume-container-devops-1`, and run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/beta`.  
      
    
3. For the second container, use image `ubuntu` with the `latest` tag only and remember to mention the tag i.e `ubuntu:latest`, container should be named as `volume-container-devops-2`, and again run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/apps`.  
      
    
4. Volume name should be `volume-share` of type `emptyDir`.  
      
    
5. After creating the pod, exec into the first container i.e `volume-container-devops-1`, and just for testing create a file `beta.txt` with any content under the mounted path of first container i.e `/tmp/beta`.  
      
    
6. The file `beta.txt` should be present under the mounted path `/tmp/apps` on the second container `volume-container-devops-2` as well, since they are using a shared volume.  
      
    

`Note:` The `kubectl` utility on `jump_host` has been configured to work with the kubernetes cluster.
```

## Crear fichero pod-share-volume.yml
```
vi pod-share-volume.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-devops
  labels:
    app: volume-share-devops
spec:
  containers:
   - name: volume-container-devops-1
     image: ubuntu:latest
     command: ["sleep", "3600"]
     volumeMounts:
       - mountPath: /tmp/beta
         name: volume-share
   - name: volume-container-devops-2
     image: ubuntu:latest
     command: ["sleep", "3600"]
     volumeMounts:
       - mountPath: /tmp/apps
         name: volume-share
         
         
  volumes:
    - name: volume-share
      emptyDir: {}
```

## Inicializar pod
```
kubectl apply -f pod-share-volume.yml
```

## Crear fichero en container y verificar que esta en ambos contenedores
```
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- bash
cd /tmp/beta && touch beta.txt && ls
kubectl exec -it volume-share-devops -c volume-container-devops-2 -- bash
cd /tmp/apps && ls
```
