# Day 51: Execute Rolling Updates in Kubernetes
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

# Day 52: Revert Deployment to Previous Version in Kubernetes
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

# Day 53: Resolve VolumeMounts Issue in Kubernetes
```
We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:



The pod name is nginx-phpfpm and configmap name is nginx-config. Identify and fix the problem.


Once resolved, copy /home/thor/index.php file from the jump host to the nginx-container within the nginx document root. After this, you should be able to access the website using Website button on the top bar.


Note: The kubectl utility on jump_host is configured to operate with the Kubernetes cluster.
```

## Inspeccionar
```
kubectl get pod,cm
kubectl get cm nginx-config -o yaml
kubectl get pod nginx-phpfpm -o yaml
```

## Editar pod
```
kubectl edit pod nginx-phpfpm
```

```
- image: php:7.2-fpm-alpine
    imagePullPolicy: IfNotPresent
    name: php-fpm-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/www/html
      name: shared-files
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-62kxj
      readOnly: true
  - image: nginx:latest
    imagePullPolicy: Always
    name: nginx-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /usr/share/nginx/html ####CAMBIAR
      name: shared-files
    - mountPath: /etc/nginx/nginx.conf
      name: nginx-config-volume
      subPath: nginx.conf
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-62kxj
      readOnly: true
```

No nos va dejar editarlo pero nos va crear un fichero temporal en **/tmp** y vamos a editar ese fichero y vamos aplicar los cambios:
```
error: pods "nginx-phpfpm" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-29276641.yaml"
error: Edit cancelled, no valid changes were saved.
```


```
cd /tmp && ls
demofile2.json
kubectl-edit-1115096616.yaml
kubectl-edit-1916071029.yaml
kubectl-edit-1943591351.yaml
kubectl-edit-2646138055.yaml
```

```
vi kubectl-edit-<hash>.yaml
```

```
#### antes ####
volumeMounts:
    - mountPath: /usr/share/nginx/html
#### despues ####
volumeMounts:
    - mountPath: /var/www/html
```

Antes de aplicar los cambios debemos eliminar el pod actual.
```
kubectl delete pod nginx-phpfpm
```

```
kubectl apply -f kubectl-edit-<hash>.yaml
```

Con este cambio nos va dar el error de que no tenemos perimiso: **403 Forbidden** 
## Configurar permisos
```
kubectl delete pod nginx-phpfpm
vi kubectl-edit-<hash>.yaml
```

```
#### antes ####
spec:
  
#### despues ####
spec:
  securityContext:
    fsGroup: 3333
```

Volvemos a eliminar y aplicar los cambios.

## Ingresar a los contenedores (opcional)
```
kubectl exec -it nginx-phpfpm -c nginx-container -- sh
kubectl exec -it nginx-phpfpm -c php-fpm-container -- sh
```

## Copiar fichero
```
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html -c nginx-container
kubectl exec -it nginx-phpfpm -c nginx-container -- sh
cd /var/www/html
chmod 666 index.php
```

# Day 54: Kubernetes Shared Volumes
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

# Day 55: Kubernetes Sidecar Containers
```
We have a web server container running the nginx image. The access and error logs generated by the web server are not critical enough to be placed on a persistent volume. However, Nautilus developers need access to the last 24 hours of logs so that they can trace issues and bugs. Therefore, we need to ship the access and error logs for the web server to a log-aggregation service. Following the separation of concerns principle, we implement the Sidecar pattern by deploying a second container that ships the error and access logs from nginx. Nginx does one thing, and it does it well—serving web pages. The second container also specializes in its task—shipping logs. Since containers are running on the same Pod, we can use a shared emptyDir volume to read and write logs.


Create a pod named webserver.

Create an emptyDir volume shared-logs.

Create two containers from nginx and ubuntu images with latest tag only and remember to mention tag i.e nginx:latest, nginx container name should be nginx-container and ubuntu container name should be sidecar-container on webserver pod.

Add command on sidecar-container "sh","-c","while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"

Mount the volume shared-logs on both containers at location /var/log/nginx, all containers should be up and running.

Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.
```

## Crear pod-volume-log.yml
```
vi pod-volume-log.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: webserver
  labels:
    app: webserver
spec:
  containers:
   - name: nginx-container
     image: nginx:latest
     volumeMounts:
       - mountPath: /var/log/nginx
         name: shared-logs
   - name: sidecar-container
     image: ubuntu:latest
     command: ["sh","-c","while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"]
     volumeMounts:
       - mountPath: /var/log/nginx
         name: shared-logs
         
         
  volumes:
    - name: shared-logs
      emptyDir: {}
```

## Inicializar pod
```
kubectl apply -f pod-volume-log.yml
```

## Verificar los logs
```
kubectl exec -it webserver -c nginx-container -- bash
curl localhost
cd /var/log/nginx
tail -f access.log
tail -f error.log
```

# Day 56: Deploy Nginx Web Server on Kubernetes Cluster
```
Some of the Nautilus team developers are developing a static website and they want to deploy it on Kubernetes cluster. They want it to be highly available and scalable. Therefore, based on the requirements, the DevOps team has decided to create a deployment for it with multiple replicas. Below you can find more details about it:


Create a deployment using nginx image with latest tag only and remember to mention the tag i.e nginx:latest. Name it as nginx-deployment. The container should be named as nginx-container, also make sure replica counts are 3.

Create a NodePort type service named nginx-service. The nodePort should be 30011.

Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.
```

## Crear deploy-nginx.yml
```
vi deploy-nginx.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30011
      protocol: TCP
```

## Iniciar objetos
```
kubectl apply -f deploy-nginx.yml
```

# Day 57: Print Environment Variables
```
The Nautilus DevOps team is working on to setup some pre-requisites for an application that will send the greetings to different users. There is a sample deployment, that needs to be tested. Below is a scenario which needs to be configured on Kubernetes cluster. Please find below more details about it.


Create a pod named print-envars-greeting.

Configure spec as, the container name should be print-env-container and use bash image.

Create three environment variables:

a. GREETING and its value should be Welcome to

b. COMPANY and its value should be Stratos

c. GROUP and its value should be Group

Use command ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"'] (please use this exact command), also set its restartPolicy policy to Never to avoid crash loop back.

You can check the output using kubectl logs -f print-envars-greeting command.


Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.
```

## Crear pod-env.yml
```
vi pod-env.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
  labels:
    app: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
   - name: print-env-container
     image: bash:latest
     env:
     - name: GREETING
       value: "Welcome to"
     - name: COMPANY
       value: "Stratos"
     - name: GROUP
       value: "Group"
     command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
     
```

## Inicializar pod
```
kubectl apply -f pod-env.yml
```

## Verificar environment
```
kubectl logs -f print-envars-greeting
```

# Day 58: Deploy Grafana on Kubernetes Cluster
```
The Nautilus DevOps teams is planning to set up a Grafana tool to collect and analyze analytics from some applications. They are planning to deploy it on Kubernetes cluster. Below you can find more details.  
  

  

1.) Create a deployment named `grafana-deployment-nautilus` using any grafana image for Grafana app. Set other parameters as per your choice.  
  

2.) Create `NodePort` type service with nodePort `32000` to expose the app.  
  

`You need not to make any configuration changes inside the Grafana app once deployed, just make sure you are able to access the Grafana login page.`  
  

`Note:` The `kubectl` on `jump_host` has been configured to work with kubernetes cluster.
```

## Crear grafana.yml
```
vi grafana.yml
```


```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana-deployment-nautilus
  name: grafana-deployment-nautilus
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      securityContext:
        fsGroup: 472
        supplementalGroups:
          - 0
      containers:
        - name: grafana
          image: grafana/grafana:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
              name: http-grafana
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /robots.txt
              port: 3000
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 30
            successThreshold: 1
            timeoutSeconds: 2
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 3000
            timeoutSeconds: 1
          resources:
            requests:
              cpu: 250m
              memory: 750Mi
          volumeMounts:
            - mountPath: /var/lib/grafana
              name: grafana-pv
      volumes:
        - name: grafana-pv
          persistentVolumeClaim:
            claimName: grafana-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
spec:
  ports:
    - port: 3000
      protocol: TCP
      targetPort: http-grafana
      nodePort: 32000
  selector:
    app: grafana
  sessionAffinity: None
  type: NodePort
```

## Inicializar objetos
```
kubectl apply -f grafana.yml
```

```
kubectl get pod
kubectl logs -f grafana-<sufijo>
```
# Day 59: Troubleshoot Deployment issues in Kubernetes
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

# Day 60: Persistent Volumes in Kubernetes
```
The Nautilus DevOps team is working on a Kubernetes template to deploy a web application on the cluster. There are some requirements to create/use persistent volumes to store the application code, and the template needs to be designed accordingly. Please find more details below:


Create a PersistentVolume named as pv-devops. Configure the spec as storage class should be manual, set capacity to 5Gi, set access mode to ReadWriteOnce, volume type should be hostPath and set path to /mnt/sysops (this directory is already created, you might not be able to access it directly, so you need not to worry about it).

Create a PersistentVolumeClaim named as pvc-devops. Configure the spec as storage class should be manual, request 3Gi of the storage, set access mode to ReadWriteOnce.

Create a pod named as pod-devops, mount the persistent volume you created with claim name pvc-devops at document root of the web server, the container within the pod should be named as container-devops using image nginx with latest tag only (remember to mention the tag i.e nginx:latest).

Create a node port type service named web-devops using node port 30008 to expose the web server running within the pod.

Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.
```

## Create volume-persistence.yml
```
vi volume-persistence.yml
```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-devops
spec:
  storageClassName: "manual"
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  hostPath:
    path: "/mnt/sysops"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-devops
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-devops
  labels:
    app: xfusion-web
spec:
  containers:
    - name: container-devops
      image: nginx:latest
      ports:
      - containerPort: 80
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: devops-storage

  volumes: 
    - name: devops-storage
      persistentVolumeClaim:  
        claimName: pvc-devops

---
apiVersion: v1
kind: Service
metadata:
  name: web-devops
spec:
  type: NodePort
  selector:
    app: xfusion-web
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
      protocol: TCP
```

## Inicializar objetos
```
kubectl apply -f volume-persistence.yml
```

## Entrar al pod
```
kubectl exec -it pod-devops -- bash
cd /usr/share/nginx/html
echo <h1>hola PIER como estas</h1> > index.html
curl IP:30008
```

# Day 61: Init Containers in Kubernetes
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