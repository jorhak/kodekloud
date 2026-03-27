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
