```
The Nautilus DevOps team is planning to set up a Jenkins CI server to create/manage some deployment pipelines for some of the projects. They want to set up the Jenkins server on Kubernetes cluster. Below you can find more details about the task:


1) Create a namespace jenkins

2) Create a Service for jenkins deployment. Service name should be jenkins-
   service under jenkins namespace, type should be NodePort, nodePort should be 
   30008

3) Create a Jenkins Deployment under jenkins namespace, It should be name as 
   jenkins-deployment , labels app should be jenkins , container name should be 
   jenkins-container , use jenkins/jenkins image , containerPort should be 8080 
   and replicas count should be 1. It should also have the environment variable 
   JAVA_OPTS with the value -Djenkins.install.runSetupWizard=false to skip the 
   initial setup wizard.

Make sure to wait for the pods to be in running state and make sure you are able to access the Jenkins UI screen in the browser before hitting the Check button.

Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```
# Crear namespace
```bash
vi namespace_jenkins.yml
```

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins
```
# Crear service
```bash
vi service_jenkins.yml
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: jenkins
spec:
  type: NodePort
  selector:
    app: jenkins
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30008
    protocol: TCP
```
# Crear deployment
```bash
vi deploy_jinkins.yml
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment
  namespace: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins-container
        image: jenkins/jenkins
        env:
          - name: JAVA_OPTS
            value: "-Djenkins.install.runSetupWizard=false"
        ports:
        - containerPort: 8080
```
# Ejecutar
```bash
kubectl apply -f .
```
# Verificar
```bash
kubectl get deploy,svc,pod
```
En la parte superior derecha presionamos el boton **APP** para ver la UI.