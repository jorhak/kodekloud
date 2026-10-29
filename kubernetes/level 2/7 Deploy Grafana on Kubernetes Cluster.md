```
The Nautilus DevOps teams is planning to set up a Grafana tool to collect and analyze analytics from some applications. They are planning to deploy it on Kubernetes cluster. Below you can find more details.  
  

  

1.) Create a deployment named `grafana-deployment-devops` using any grafana
image for Grafana app. Set other parameters as per your choice.  
  

2.) Create `NodePort` type service with nodePort `32000` to expose the app.  
  

`You do not need to make any configuration changes inside the Grafana app once deployed; just make sure you can access the Grafana login page.`  
  

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.
```
# Crear namespace (OPCIONAL)
```bash
kubectl create namespace my-grafana
```
### Verificar que se creo namespace (OPCIONAL)
```bash
kubectl get namespace my-grafana
```
# Crear deploy.yml
```bash
vi deploy_grafana.yml
```

```yml
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
    app: grafana
  name: grafana-deployment-datacenter
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
# Ejecutar
```bash
kubectl apply -f deploy_grafana.yml
```
#### Si creamos namespace
```bash
kubectl apply -f deploy_grafana.yml -n my-grafana
```
# Verificar
```bash
kubectl get all
```

```bash
kubectl describe deploy grafana-deployment-devops
```

```bash
kubectl describe pod grafana-deployment-devops-<reemplazar sufijo>
```
#### Si creamos namespce
```bash
kubectl get all -n my-grafana
```

```bash
kubectl describe deploy grafana-deployment-devops -n my-grafana
```

```bash
kubectl describe pod grafana-deployment-devops-<reemplazar sufijo> -n my-grafana
```
