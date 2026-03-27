```
Last week, the Nautilus DevOps team deployed a redis app on Kubernetes cluster, which was working fine so far. This morning one of the team members was making some changes in this existing setup, but he made some mistakes and the app went down. We need to fix this as soon as possible. Please take a look.



The deployment name is redis-deployment. The pods are not in running state right now, so please look into the issue and fix the same.


Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.
```

## Inspeccionar 
```
kubectl get deploy,pod
kubectl describe deploy redis-deployment
kubectl describe pod redis-deployment-<hash>
```

En el **pod** pudimos ver que no encuentra el objeto
```
Type     Reason       Age                 From               Message
  ----     ------       ----                ----               -------
  Normal   Scheduled    17m                 default-scheduler  Successfully assigned default/redis-deployment-54cdf4f76d-dt78d to kodekloud-control-plane
  Warning  FailedMount  108s (x7 over 15m)  kubelet            Unable to attach or mount volumes: unmounted volumes=[config], unattached volumes=[], failed to process volumes=[]: timed out waiting for the condition
  Warning  FailedMount  68s (x16 over 17m)  kubelet            MountVolume.SetUp failed for volume "config" : configmap "redis-conig" not found <<<<<AQUI
```

Ahora vamos a listar los **configmap**
```
kubectl get cm
```

```
NAME               DATA   AGE
kube-root-ca.crt   1      31m
redis-config       2      19m
```

Efectivamente podemos ver que no existe un **configmap** "redis-conig", esta mal escrito el nombre del **configmap**.
## Editar el contenido del deploy
```
kubectl edit deploy redis-deployment
```

```
#### AFTER ####
volumes:
      - emptyDir: {}
        name: data
      - configMap:
          defaultMode: 420
          name: redis-conig
          
#### BEFORE ####
volumes:
      - emptyDir: {}
        name: data
      - configMap:
          defaultMode: 420
          name: redis-config
```

Volvemos a ver el estado de los objetos
```
kubectl get deploy,pod
```

```
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis-deployment   0/1     1            0           23m

NAME                                    READY   STATUS              RESTARTS   AGE
pod/redis-deployment-54cdf4f76d-dt78d   0/1     ContainerCreating   0          23m
pod/redis-deployment-5bcd4c7d64-gxnlz   0/1     ErrImagePull        0          14s
```

Aqui podemos ver que nos salio un nuevo error con el **pod** no pudo descargar la imagen, vamos a ver la descripcion del **pod**.
```
kubectl describe pod redis-deployment-5bcd4c7d64-gxnlz
```

```
Normal   Scheduled  2m55s                default-scheduler  Successfully assigned default/redis-deployment-5bcd4c7d64-gxnlz to kodekloud-control-plane
  Normal   Pulling    81s (x4 over 2m54s)  kubelet            Pulling image "redis:alpin"  <<<< AQUI
  Warning  Failed     81s (x4 over 2m54s)  kubelet            Failed to pull image "redis:alpin": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/redis:alpin": failed to resolve reference "docker.io/library/redis:alpin": docker.io/library/redis:alpin: not found
  Warning  Failed     81s (x4 over 2m54s)  kubelet            Error: ErrImagePull
  Warning  Failed     69s (x6 over 2m53s)  kubelet            Error: ImagePullBackOff
  Normal   BackOff    55s (x7 over 2m53s)  kubelet            Back-off pulling image "redis:alpin" <<<<< AQUI
```
El nombre de la imagen esta mal escrito.
## Editar el contenido del deploy
```
kubectl edit deploy redis-deployment
```

```
### AFTER ###
containers:
- image: redis:alpin
### BEFORE ###
containers:
- image: redis:alpine
```

Volvemos a ver el estado de los objetos
```
kubectl get deploy,pod
```

Finalmente queda todo bien
```
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis-deployment   1/1     1            1           29m

NAME                                    READY   STATUS    RESTARTS   AGE
pod/redis-deployment-7c8d4f6ddf-wlt6z   1/1     Running   0          74s
```
