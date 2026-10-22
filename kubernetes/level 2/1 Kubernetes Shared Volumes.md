```text
We are working on an application that will be deployed on multiple containers within a pod on Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario. Below you can find more details about it.



Create a pod named volume-share-xfusion.


For the first container, use image ubuntu with latest tag only and remember to mention the tag i.e ubuntu:latest, container should be named as volume-container-xfusion-1, and run a sleep command for it so that it remains in running state. Volume volume-share should be mounted at path /tmp/official.


For the second container, use image ubuntu with the latest tag only and remember to mention the tag i.e ubuntu:latest, container should be named as volume-container-xfusion-2, and again run a sleep command for it so that it remains in running state. Volume volume-share should be mounted at path /tmp/apps.


Volume name should be volume-share of type emptyDir.


After creating the pod, exec into the first container i.e volume-container-xfusion-1, and just for testing create a file official.txt with the content Welcome to xFusionCorp Industries under the mounted path of first container i.e /tmp/official.


The file official.txt should be present under the mounted path /tmp/apps on the second container volume-container-xfusion-2 as well, since they are using a shared volume.


Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```
# Crear pod
```bash
vi pod.yml
```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-xfusion
  labels:
    estado: "desarrollo"
spec:
  containers:
    - name: volume-container-xfusion-1
      image: ubuntu:latest
      command: ["/bin/sh", "-c", "while true; do sleep 30; done"]
      volumeMounts:
        - mountPath: /tmp/official
          name: volume-share
    - name: volume-container-xfusion-2
      image: ubuntu:latest
      command: ["/bin/sh", "-c", "while true; do sleep 30; done"]
      volumeMounts:
        - mountPath: /tmp/apps
          name: volume-share
  volumes:
    - name: volume-share
      emptyDir: {}
```
## Ejecutar
```bash
kubectl apply -f pod.yml
kubectl get pod
kubectl describe pod volume-share-xfusion
```
## Ingresar al container volume-container-devops-1
Vamos a crear un fichero
```bash
kubectl exec -it volume-share-xfusion -c volume-container-xfusion-1 -- /bin/sh
```

```bash
echo "Welcome to xFusionCorp Industries" > /tmp/official/official.txt
```
## Ingresar al container volume-container-devops-2
Vamos a ver que en este contenedor vamos a tener el fichero **beta.txt**, esto porque estamos compartiendo el volumen.
```bash
kubectl exec -it volume-share-xfusion -c volume-container-xfusion-2 -- /bin/sh
ls /tmp/apps
cat /tmp/apps/official.txt
```
De esta forma hemos terminado la tarea.
## Eliminar pod (OPCIONAL)
```bash
kubectl delete -f pod.yml
```
Esto lo podemos utilizar si necesitamos que otro contenedor utilice lo que genera el otro contendor para seguir en la tarea, por ejemplo sonarqube uno en donde este la UI y en el otro donde se ejecutan los comandos de analisis, y de esta forma se genera el reporte, en el segundo contenedor debe estar el codigo que se va analizar.