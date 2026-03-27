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