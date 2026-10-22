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