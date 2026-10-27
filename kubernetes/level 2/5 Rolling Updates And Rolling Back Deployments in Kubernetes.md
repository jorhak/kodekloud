```
There is a production deployment planned for next week. The Nautilus DevOps team wants to test the deployment update and rollback on Dev environment first so that they can identify the risks in advance. Below you can find more details about the plan they want to execute.



1. Create a namespace devops. Create a deployment called httpd-deploy under this 
   new namespace, It should have one container called httpd, use httpd:2.4.28 
   image and 4 replicas. The deployment should use RollingUpdate strategy with 
   maxSurge=1, and maxUnavailable=2. Also create a NodePort type service named 
   httpd-service and expose the deployment on nodePort: 30008.


2. Now upgrade the deployment to version httpd:2.4.43 using a rolling update.


3. Finally, once all pods are updated undo the recent update and roll back to 
   the previous/original version.


Note:

	a. The kubectl utility on the jump-host has been configured to work with the 
	Kubernetes cluster.


	b. Please make sure you only use the specified image(s) for this deployment 
	and as per the sequence mentioned in the task description. If you mistakenly 
	use a wrong image and fix it later, that will also distort the revision 
	history which can eventually fail this task.
```
# Crear namespace
```bash
vi namespace.yml
```

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: datacenter
```
## Ejecutar namespace
```bash
kubectl apply -f namespace.yml
```
# Crear deploy.yml
```bash
vi deploy.yml
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deploy
  namespace: datacenter
spec:
  replicas: 4
  selector:
    matchLabels:
      app: httpd
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.27
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
  namespace: datacenter
spec:
  type: NodePort
  selector:
    app: httpd
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30008
    protocol: TCP

```
## Ejecutar deploy
```bash
kubectl apply -f deploy.yml
```
## Verificar
```bash
kubectl get deploy,svc,pod -n datacenter
kubectl describe pod/httpd-deploy-<reemplazar> -n datacenter
```
# Acutalizar imagen
```bash
kubectl set image deployment/httpd-deploy httpd=httpd:2.4.43 -n datacenter
```
## Verificar
```bash
kubectl get deploy,svc,pod -n datacenter
kubectl describe pod/httpd-deploy-<reemplazar> -n datacenter
```
# Roolback
```bash
kubectl rollout status deploy httpd-deploy -n datacenter
kubectl rollout history deploy httpd-deploy -n datacenter
kubectl rollout undo deploy httpd-deploy --to-revision=1 -n datacenter
```
# Verificar que volvio al original
```bash
kubectl get deploy,svc,pod -n datacenter
kubectl describe pod/httpd-deploy-<reemplazar> -n datacenter
```
