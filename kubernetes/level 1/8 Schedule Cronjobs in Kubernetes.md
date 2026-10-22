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