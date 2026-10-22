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

```yaml
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