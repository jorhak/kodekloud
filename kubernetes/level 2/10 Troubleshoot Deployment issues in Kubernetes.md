```
Last week, the Nautilus DevOps team deployed a redis app on Kubernetes cluster, which was working
fine so far. This morning one of the team members was making some changes in this existing setup, but he
made some mistakes and the app went down. We need to fix this as soon as possible. Please take a look.

The deployment name is redis-deployment. The pods are not in running state right now, so please look
into the issue and fix the same.

Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```
# Listar deploy y pod
```bash
kubectl get deploy,pod
```
# Describir pod
```bash
kubectl describe pod/redis-deployment-6bc546f779-9flqz
```

```text
Type     Reason       Age                From               Message
----     ------       ----               ----               -------
Normal   Scheduled    5m                 default-scheduler  Successfully assigned default/redis-deployment-6bc546f779-9flqz to jump-host

Warning  FailedMount  50s (x10 over 5m)  kubelet            MountVolume.SetUp failed for volume "config" : configmap "redis-conig" not found
```
Nos indica que no encuentra configmap: **redis-conig**. Vamos a listar configmap para ver si existe el objeto.
## Listar configmap
```bash
kubectl get cm
```

```text
NAME               DATA   AGE
kube-root-ca.crt   1      66m
redis-config       2      8m39s
```
Podemos ver que solo existe **redis-config** y no **reids-conig**.
# Editar deploy
```bash
kubectl edit deployment.apps/redis-deployment
```
#### Before
```yaml
      - configMap:                                  
          defaultMode: 420                          
          name: redis-conig                         
        name: config
```
#### After
```yaml
      - configMap:                                  
          defaultMode: 420                          
          name: redis-config                         
        name: config
```
# Verificar
```bash
kubectl get deploy,pod
```
Ahora tenemos un nuevo error.
# Describir pod
```bash
kubectl describe pod/redis-deployment-85cd7f84f5-2qrvg
```

```text
  Normal   Pulling    54s (x5 over 3m53s)   kubelet            Pulling image "redis:alpin"

  Warning  Failed     53s (x5 over 3m52s)   kubelet            Failed to pull image "redis:alpin": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/redis:alpin": failed to resolve reference "docker.io/library/redis:alpin": docker.io/library/redis:alpin: not found

  Warning  Failed     53s (x5 over 3m52s)   kubelet            Error: ErrImagePull

  Normal   BackOff    14s (x13 over 3m51s)  kubelet            Back-off pulling image "redis:alpin"

  Warning  Failed     14s (x13 over 3m51s)  kubelet            Error: ImagePullBackOff
```
Vemos que tenemos problemas al descargar la imgen **redis:alpin**, podemos suponer que el nombre de la imagen esta mal escrito.
# Editar deploy
```bash
kubectl edit deployment.apps/redis-deployment
```
#### Before
```yaml
      containers:                          
      - image: redis:alpin
```
#### After
```yaml
      containers:                          
      - image: redis:alpine
```
# Verificar
```bash
kubectl get deploy,pod
```