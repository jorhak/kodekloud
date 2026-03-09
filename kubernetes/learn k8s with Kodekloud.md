# 4 Set Resource Limits in Kubernetes Pods
```
The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints. To address this, they plan to set limits on resource utilization. Here are the details:


Create a pod named httpd-pod with a container named httpd-container. Use the httpd image with the latest tag (specify as httpd:latest). Set the following resource limits:

Requests: Memory: 15Mi, CPU: 100m

Limits: Memory: 20Mi, CPU: 100m

Note: The kubectl utility on jump_host is configured to operate with the Kubernetes cluster.
```

1. Crear **Pod**:
```
vi pod.yaml
```

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
```

2. Ejecutar **Pod**:
```
kubectl apply -f pod.yaml
```

3. Verificar **Pod**:
```
kubectl get pod httpd-pod
```

4. Eliminar **Pod**:
```
kubectl delete -f pod.yaml
```

# 5 Execute Rolling Updates in Kubernetes
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

# 6 Revert Deployment to Previous Version in Kubernetes
```
Earlier today, the Nautilus DevOps team deployed a new release for an application. However, a customer has reported a bug related to this recent release. Consequently, the team aims to revert to the previous version.


There exists a deployment named nginx-deployment; initiate a rollback to the previous revision.

Note: The kubectl utility on jump_host is configured to interact with the Kubernetes cluster.
```

1. Describir **deployment**:
```
kubectl describe deployment/nginx-deployment
kubectl get deployment/nginx-deployment -o yaml
```

2. Ver historial 
```
kubectl rollout history deployment/nginx-deployment
```

2. Ver estado
```
kubectl rollout status deployment/nginx-deployment
```

3. Rollback
```
kubectl rollout undo deployment/nginx-deployment
## si queremos especificar
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

# 7 Deploy ReplicaSet in Kubernetes Cluster
```
The Nautilus DevOps team is gearing up to deploy applications on a Kubernetes cluster for migration purposes. A team member has been tasked with creating a ReplicaSet outlined below:



Create a ReplicaSet using httpd image with latest tag (ensure to specify as httpd:latest) and name it httpd-replicaset.


Apply labels: app as httpd_app, type as front-end.


Name the container httpd-container. Ensure the replica count is 4.


Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```

1. Crear **Replicaset**:
```code
nano httpd-replicaset.yaml
```


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: httpd-replicaset 
  labels:
    app: httpd_app
    type: front-end
spec:
  # numero de replicas
  replicas: 4
  # selector para gestionar los pods para esta ReplicaSet
  selector:
    matchLabels:
      tier: frontend
  # template para crear nuevos pods
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: httpd-container
        image: httpd:latest
        ports:
        - containerPort: 80
```

2. Ejecutar
```
kubectl apply -f httpd-replicaset.yaml
```

3. Verificar
```
kubectl get pod,rs
###Describir replicaset
kubectl describe rs/httpd-replicaset
```

# 8 Schedule Cronjobs in Kubernetes
```
The Nautilus DevOps team is setting up recurring tasks on different schedules. Currently, they're developing scripts to be executed periodically. To kickstart the process, they're creating cron jobs in the Kubernetes cluster with placeholder commands. Follow the instructions below:



Create a cronjob named xfusion.


Set Its schedule to something like */7 * * * *. You can set any schedule for now.


Name the container cron-xfusion.


Utilize the httpd image with latest tag (specify as httpd:latest).


Execute the dummy command echo Welcome to xfusioncorp!.


Ensure the restart policy is OnFailure.


Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```

1. Crear **CronJob**:
```
nano cron.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: xfusion
spec:
  schedule: "*/7 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cron-xfusion
            image: httpd:latest
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure

```

2. Ejecutar
```
kubectl apply -f cron.yaml
```

3. Estado del **cron**:
```
kubectl get cronjob xfusion
kubectl get jobs --watch
```

4. Verificar que se esta ejecutando
```
pods=$(kubectl get pods --selector=job-name=<pod> --output=jsonpath={.items[*].metadata.name})
```

```
kubectl logs $pods
```

# 9 Create Countdown Job in Kubernetes
```
The Nautilus DevOps team is crafting jobs in the Kubernetes cluster. While they're developing actual scripts/commands, they're currently setting up templates and testing jobs with dummy commands. Please create a job template as per details given below:


Create a job named countdown-xfusion.

The spec template should be named countdown-xfusion (under metadata), and the container should be named container-countdown-xfusion

Utilize image debian with latest tag (ensure to specify as debian:latest), and set the restart policy to Never.

Execute the command sleep 5

Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```

1. Crear **job**:
```code
nano job.yaml
```

```yaml
apiVersion: batch/v1 
kind: Job 
metadata:
  name: countdown-xfusion
spec:
  template:
    metadata:
      name: countdown-xfusion
    spec:
      containers:
      - name: container-countdown-xfusion
        image: debian:latest
        command: ["sleep", "5"]
      restartPolicy: Never
```

2. Ejecutar
```code
kubectl apply -f job.yaml
```

3. Veriricar
```code
kubectl get job/countdown-xfusion
kubectl get pods
```

4. Eliminar **job**
```code
kubectl delete -f job.yaml
```

# 10 Set Up Time Check Pod in Kubernetes
```
The Nautilus DevOps team needs a time check pod created in a specific Kubernetes namespace for logging purposes. Initially, it's for testing, but it may be integrated into an existing cluster later. Here's what's required:

  
  

1. Create a pod called `time-check` in the `nautilus` namespace. The pod should contain a container named `time-check`, utilizing the `busybox` image with the `latest` tag (specify as `busybox:latest`).
    
2. Create a config map named `time-config` with the data `TIME_FREQ=4` in the same namespace.
    
3. Configure the `time-check` container to execute the command: `while true; do date; sleep $TIME_FREQ;done`. Ensure the result is written `/opt/devops/time/time-check.log`. Also, add an environmental variable `TIME_FREQ` in the container, fetching its value from the config map `TIME_FREQ` key.
    
4. Create a volume `log-volume` and mount it at `/opt/devops/time` within the container.
    

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.
```
1. Ver namespace
```
kubectl get ns
```

- Crear namespace datacenter
```
kubectl create ns datacenter
```

- Opcional colocar namespace por defecto datacenter
```
kubectl config set-context --current --namespace=datacenter
```

- Verificar que se cambio namespace
```
kubectl config view --minify | grep namespace
```

2. Crear configmap

```
vi config.yaml
```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-config
  namespace: datacenter
data:
  TIME_FREQ: "4"
```

```
kubectl apply -f config.yaml
```

3. Crear pod
- Crear directorio
```
mkdir /home/thor/susana
```

```
vi pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: datacenter
spec:
  containers:
    - name: time-check
      image: busybox:latest
      command: ["/bin/sh", "-c"]
      args: ["while true; do date >> /opt/dba/time/time-check.log; sleep $TIME_FREQ;done"]
      env:
        - name: TIME_FREQ
          valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
      volumeMounts:
      - mountPath: "/opt/dba/time"
        name: log-volume
  volumes:
  - name: log-volume
    hostPath:
      path: /home/thor/susana
      type: Directory
```

```
kubectl apply -f pod.yaml
```

- Describir pod
```
kubectl describe pod/time-check
```

- Verificar pod
```
kubectl get pod/time-check
```

4. Verificar volumen
- En host
```
cd /home/thor/susana/
cat time-check.log
```

- En el contenedor
```
kubectl exec pod/time-check -- cat /opt/dba/time/time-check.log
```
4. 