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

# Day 70: Configure Jenkins User Access
```
The Nautilus team is integrating Jenkins into their CI/CD pipelines. After setting up a new Jenkins server, they're now configuring user access for the development team, Follow these steps:  
  
1. Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login with username `admin` and password `Adm!n321`.

2. Create a jenkins user named `kareem` with the password`LQfKeWWxWD`. Their full name should match `Kareem`.  
  
3. Utilize the `Project-based Matrix Authorization Strategy` to assign `overall read` permission to the `kareem` user.  
  
4. Remove all permissions for `Anonymous` users (if any) ensuring that the `admin` user retains overall `Administer` permissions.  
  
5. For the existing job, grant `kareem` user only `read` permissions, disregarding other permissions such as Agent, SCM etc.  
  
`Note:`  
  
6. You may need to install plugins and restart Jenkins service. After plugins installation, select `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page.  

  
7. After restarting the Jenkins service, wait for the Jenkins login page to reappear before proceeding. Avoid clicking `Finish` immediately after restarting the service.  
  
  
8. Capture screenshots of your configuration for review purposes. Consider using screen recording software like `loom.com` for documentation and sharing.
```

# Crear usuario
Despues de haber ingresado damos click en **Manage Jenkins**>**Users**>**Create User**, luego rellenamos el formulario con los datos provistos. Y pulsamos el boton **Create User**.

Luego actualizamos los _pluigns_ que estan desactualizados y reiniciamos **Jenkins**. 

Ahora debemos instalar el _plugin_ necesario para los permisos. Estamos en **Dashboard**, damos click en **Manage Jenkins**>**Plugins**>**Available plugins** y en la barra de busqueda escribimos **Matrix Authorization**, marcamos esta opcion y damos click en el boton **Install**. Al igual que en la actualizacion de los _plugins_ vamos a marcar el check de reinicio una ves se termine la instalacion del _plugin_.

Con el plugin instalado procedemos con los siguientes pasos, vamos a configurar al usuario **kareen** con los permisos que son requeridos. Para ellos nos ubicamos en **Dashboard**, damos click en **Manage Jenkins**>**Security**, buscamos el apartado **Authorization** y desplegamos la opciones y seleccionamos **Project-based Matrix Authorization Strategy**, agregamos al usuario _kareem_, damos click en el boton **Add user...** y luego introducimos el **User ID** que seria _kareem_ y damos click en el boton **OK**. Finalmente damos click en **Save**.

Siguiendo con el siguiente paso vamos a configurar el proyecto para que el usuario **siva** solo tenga permiso: **READ**.
Nos ubicamos en **Dashboard** luego damos click en **Helloworld**>**Configure** marcamos **Enable project-based security**, al igual que en el paso previo vamos a agregar a nuestro usuario **kareem**. Y marcamos en el area **Job**>**Read**

# Day 71: Configure Jenkins Job for Package Installation
```
Some new requirements have come up to install and configure some packages on the Nautilus infrastructure under Stratos Datacenter. The Nautilus DevOps team installed and configured a new Jenkins server so they wanted to create a Jenkins job to automate this task. Find below more details and complete the task accordingly:  
  

  

1. Access the Jenkins UI by clicking on the `Jenkins` button in the top bar. Log in using the credentials: username `admin` and password `Adm!n321`.  
  

2. Create a new Jenkins job named `install-packages` and configure it with the following specifications:  
  

- Add a string parameter named `PACKAGE`.
- Configure the job to install a package specified in the `$PACKAGE` parameter on the `storage server` within the `Stratos Datacenter`.  
      
    

`Note`:  
  

1. Ensure to install any required plugins and restart the Jenkins service if necessary. Opt for `Restart Jenkins when installation is complete and no jobs are running` on the plugin installation/update page. Refresh the UI page if needed after restarting the service.  
  

2. Verify that the Jenkins job runs successfully on repeated executions to ensure reliability.  
  

3. Capture screenshots of your configuration for documentation and review purposes. Alternatively, use screen recording software like `loom.com` for comprehensive documentation and sharing.
```

Luego de ingresar a **Jenkins** con la credenciales proporcionadas damos click en **Create a job** y colocamos el nombre que nos asignaron _install-packages_ y seleccionamos **Freestyle project**, luego damos click en el boton **OK**.

Actualizamos los _plugins_ que necesitan ser actualizados y reiniciamos **Jenkins**.
Instalamos los _plugins_ **SSH y 1Password Secrets** y reiniciamos **Jenkins**.

Debemos agregar los credenciales del servidor donde vamos a ejecutar los comandos. Nos encontramos en **Dashboard** damos click en **Manage Jenkins** nos dirigimos en el apartado de **Security** y damos click en **Credentials**>**global**, luego damos click en el boton **Add Credentials**.
En _Kind_ lo dejamos en **Username with password** agregamos el usuario que tenemos para ingresar **storage server** rellenamos el formulario y damos click en el boton **Create**.

Vamos a realizar lo mismo solo que vamos a cambiar _Kind_: Secret text, y rellemanos el formulario, el secreto debe ser la contrasena del usuario en donde vamos a ejecutar los comandos.

Agregamos el servidor en donde vamos a ejecutar el comando. Estamos en **Dashboard**>**Manage Jenkins**>**System** nos  dirigimos al apartado _SSH remote hosts_ y damos click en **Add**, en **hostname** agregamos la **IP**: 172.16.238.15 el **puerto**: 22 y los credenciales que va usar en nuestro caso es **natasha (storage server)**, y finalmente damos click en **Save**.
De igual modo nos vamos al partado de _1Password secret_ y agregamos la **IP**.

Nos ubicamos en **Dashboard** damos click en **install-package**>**Configure** marcamos **This project is parameterized**>**Add Parameter**>**String Parameter** y rellenamos el formulario con los parametros asignados.

Luego nos dirigimos a **Build Steps** damos click en **Add build step**>**Execute shell script on remote host using ssh**

Despues nos dirigimos al apartado de **Environment** y marcamos _Use secret text(s) or file(s)_ rellenamos el formulario, de igual modo marcamos _Execute shell script on remote host using ssh_ y rellenamos el formulario.
!image[image](hola)


# Day 72: Jenkins Parameterized Builds
```
A new DevOps Engineer has joined the team and he will be assigned some Jenkins related tasks. Before that, the team wanted to test a simple parameterized job to understand basic functionality of parameterized builds. He is given a simple parameterized job to build in Jenkins. Please find more details below:



Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and password Adm!n321.


1. Create a parameterized job which should be named as parameterized-job


2. Add a string parameter named Stage; its default value should be Build.


3. Add a choice parameter named env; its choices should be Development, Staging and Production.


4. Configure job to execute a shell command, which should echo both parameter values (you are passing in the job).


5. Build the Jenkins job at least once with choice parameter value Development to make sure it passes.


Note:

1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.


2. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

Despues de ingresar a **Jenkins** damos click en **Create job** y colocamos el nombre _parameterized-job_ y seleccionamos **Freestyle**.

Luego marcamos _This project is parameterized_ y damos click en **Add Parameter** luego seleccionamos _String Parameter_ y rellenamos el formulario.
Ahi mismo damos click en **Add Parameter** y seleccionamos _Choice Parameter_ y rellenamos el formulario

Nos dirigimos al apartado **Build Steps** y damos click en **Add build step** y seleccionamos _Execute shell_.

Finalmente damos click en **Save**.

Para ejecutarlo nos vamos a **Dashboard>parameterized job** y damos click en _Build with Parameters_, vamos a tener los parametros **Stage y env** en donde vamos a poder moficar y seleccionar una opcion respectivamente. Luego damos click sobre el boton **Build**.

# Day 73: Jenkins Scheduled Jobs
```
The devops team of xFusionCorp Industries is working on to setup centralised logging management system to maintain and analyse server logs easily. Since it will take some time to implement, they wanted to gather some server logs on a regular basis. At least one of the app servers is having issues with the Apache server. The team needs Apache logs so that they can identify and troubleshoot the issues easily if they arise. So they decided to create a Jenkins job to collect logs from the server. Please create/configure a Jenkins job as per details mentioned below:



Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and password Adm!n321

1. Create a Jenkins jobs named copy-logs.

2. Configure it to periodically build every 5 minutes to copy the Apache logs (both access_log and error_logs) from App Server 3 (from default logs location) to location /usr/src/finance on Storage Server.

Note:

1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case please make sure to refresh the UI page.

2. Please make sure to define you cron expression like this */10 * * * * (this is just an example to run job every 10 minutes).

3. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

Luego de ingresar a **Jenkins** damos click en **Create job** y le asignamos el nombre provisto y seleccionar **Freestyle project**.


Despues nos vamos a actualizar los _plugins_, seguido de eso vamos a instalar los _plugins_: **SSH y 1Password Secret** nos dirigimos a **Dashboard>Manage Jenkins>Plugins** damos click en _Available plugins_.

Seguimos con el siguiente paso que va ser el de crear las credenciales de los servidores: _App Server 3 y Storage Server_. Para ello debemos estar ubicamdos en **Dashboard>Manage Jenkins>Credentials** y damos click sobre **global** luego damos click sobre el boton **Add Credentials**. En _Kind_ lo dejamos en **Username with password**, luego rellenamos el formulario con el usuario y contrasena del servidor App 3 y por ultimo damos click en el boton **Create**. Del mismo modo para el servidor de Almacenamiento.

Despues de todo esto vamos a crear **Secret text** para ambos servidores que en este caso van a ser las contrasenas. En _Kind_ vamos a seleccionar **Secret text**, en **Secret** debemos introducir nuestro secreto que va ser la contrasena del servidor, luego damos click en el boton **Create**.

Ahora nos dirigimos a **Dashboard>Manage Jenkins>System** en el apartado _SSH remote hots_ damos click en **Add**. En _Hostname_ colocamos la **IP** del servidor App 1 y en **Port** ingresamos 22 y en Credentials selecionamos el credencial previo que creamos en este caso es _banner (servidor apache)_ damos click sobre el boton **Save**.
Lo mismo vamos hacer para el servidor de almacenamiento.

Seguimos con el siguiente paso que va ser ir a **Dashboard** y damos click sobre _copy-logs_ y luego damos click sobre **Configure**, luego nos dirigimos al apartado **Triggers** y marcamos _Build periodically_ y colocamos:
```
H/5 * * * *
```

Luego nos vamos al apartado **Environment** y marcamos _Use secret text(s) or file(s)_, para ambos servidores: App 3 y Storage Server, damos click sobre **Add** y selecionamos **Secret text**, en _Variable_ le vamos a colocar PASS_APP y en _Credentials_ le vamos asignar **servidor apache**. Y para el otro servidor hacemos lo mismo solo que cambiamos _Variable_ con PASS_STORAGE y _Credentials_ con **servidor de almacenamiento**.

En el apartado de **Environment** marcamos tambien _Execute shell script on remote host using ssh_ y en **SSH site** seleccionamos _natasha@IP_. En _Pre build script_ colocamos:
```
echo $PASS_APP | sudo -S yum install sshpass
ls /var/log/httpd/
sshpass -p $PASS_STORAGE scp -o StrictHostKeyChecking=no /var/log/httpd/access_log natasha@172.16.238.15:/usr/src/finance/
sshpass -p $PASS_STORAGE scp -o StrictHostKeyChecking=no /var/log/httpd/error_log natasha@172.16.238.15:/usr/src/finance/
```



Ahora nos dirigimos al apartado **Build Steps**, damos click sobre _Add build step_ y seleccionamos **Execute shell script on remote host using ssh**, en _SSH site_ seleccionamos natasha@IP y colocamos los comandos:
```
ls -la /usr/src/finance
```

Finalmente damos click sobre el boton **Save**.



echo $PASS_APP | sudo -S yum install sshpass -y
ls /var/log/httpd/
sshpass -p $PASS_STORAGE scp -o StrictHostKeyChecking=no /var/log/httpd/access_log natasha@172.16.238.15:/usr/src/finance/


echo $PASS_STORAGE | sudo -S chmod -R 777 /usr/src/finance

ls -la /usr/src/finance

# Day 74: Jenkins Database Backup Job
```
There is a requirement to create a Jenkins job to automate the database backup. Below you can find more details to accomplish this task:



Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and password Adm!n321.


Create a Jenkins job named database-backup.


Configure it to take a database dump of the kodekloud_db01 database present on the Database server in Stratos Datacenter, the database user is kodekloud_roy and password is asdfgdsd.


The dump should be named in db_$(date +%F).sql format, where date +%F is the current date.

Copy the db_$(date +%F).sql dump to the Backup Server under location /home/clint/db_backups.


Further, schedule this job to run periodically at */10 * * * * (please use this exact schedule format).


Note:


You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case please make sure to refresh the UI page.


Please make sure to define you cron expression like this */10 * * * * (this is just an example to run job every 10 minutes).


For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
```

Luego de ingresar a **Jenkins** lo que hacemos es dar click sobre **Create a job**, despues ingresamos el nombre propuesto y seleccionamos **Freestyle project** y presionamos **OK**.

Seguido de eso vamos a actulizar los _plugins_ y a instalar los plugins **SSH y 1Password Secrets**.
Por lo cual debemos estar en **Dashboard>Manage Jenkins>Plugins** y en _Updates_ actualizamos los plugins. Luego vamos a instalar los plugins ya mencionados y para ello nos vamos a _Available plugins_ y buscamos en la barra de busqueda.

El siguiente paso sera crear _secret text_ de: Database server, Backup server y MariaDB password.
Por lo cual nos dirigimos a **Dashboard>Manage Jenkins>Credentials** damos click sobre _global_, seguido damos click en **Add Credentials** y vamos seleccionar en _Kind_:Secret text y rellenamos el formulario. Los secretos van a ser las contrasenas de los usuarios y la contrasena para acceder a la base de datos. 

Ahora vamos a configurar el **job**, debemos estar ubicados en **Dashboard** y dar click sobre _database-backup_ y luego damos click en **Configure**.
Nos dirigimos al apartado de _Triggers_ y marcamos _Build periodically_ en donde debemos ingresar:
```
*/10 * * * *
```

Vamos a utilizar los secrets text como variables nos dirigimos al apartado de **Environment** y marcamos _Use secret text(s) or file(s)_ luego damos click en **Add** y seleccionamos _Secret text_ y rellenamos el formulario tanto para **db (PASS_DB), backup (PASS_BACKUP) y mariadb (PASS_MARIADB)**.

Luego nos vamos al apartado **Build Steps** y damos click sobre _Add build step_ y seleccionamos _Execute shell_, lo primero que vamos hacer es crear una variable para utilizarla como nombre del backup de nuestra DB, luego vamos a ejecutar el comando que se va ejecutar en el servidor **DB** que nos va generar el backup, finalmente vamos a copiarlo en el servidor **Backup**. Para no tener el fichero .sql en nuestro _WORKSPACE_ lo vamos a eliminar ya que lo vamos a copiar en otro servidor, el ultimo comando verifica que se copio el fichero en el servidor **Backup**:
```
DB_FILE="db_$(date +%F).sql"

sshpass -p "$PASS_DB" ssh -o StrictHostKeyChecking=no peter@172.16.239.10 "mariadb-dump -u kodekloud_roy --password='$PASS_MARIADB' kodekloud_db01" > "$DB_FILE"
ls -la

sshpass -p "$PASS_BACKUP" scp -o StrictHostKeyChecking=no "$DB_FILE" clint@172.16.238.16:/home/clint/db_backups

rm "$DB_FILE"

sshpass -p "$PASS_BACKUP" ssh -o StrictHostKeyChecking=no clint@172.16.238.16 "ls -la /home/clint/db_backups"
```

Finalmente damos click en **Save**
Verificamos la ejecucion presionando **Build Now**

# Day 75: Jenkins Slave Nodes
```

```