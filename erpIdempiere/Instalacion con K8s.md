# Crear persistencia
Lo primero que vamos a crear seran los volumenes (persistencia de datos):
```
nano pvc.yml
```

```
kind: PersistentVolumeClaim
metadata:
  name: postgres-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: idempiere-config-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

# Deploy y Services de Postgres
```
nano deploy_postgres.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16
        env:
        - name: POSTGRES_PASSWORD
          value: "postgres"
        - name: POSTGRES_USER
          value: "idempiere"
        - name: POSTGRES_DB
          value: "idempiere"
        - name: TZ
          value: "America/La_Paz"
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: db-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: db-storage
        persistentVolumeClaim:
          claimName: postgres-data-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

# Deploy y Services de Idempiere
```
nano deploy_idempiere.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: idempiere
spec:
  replicas: 1
  selector:
    matchLabels:
      app: idempiere
  template:
    metadata:
      labels:
        app: idempiere
    spec:
      containers:
      - name: idempiere
        image: idempiereofficial/idempiere:12-master
        env:
        - name: TZ
          value: "America/La_Paz"
        # iDempiere suele necesitar saber dónde está la DB. 
        # Asegúrate de que en su config use 'postgres' como host.
        ports:
        - containerPort: 8080
        - containerPort: 8443
        - containerPort: 12612
        volumeMounts:
        - name: config-v
          mountPath: /opt/idempiere/configuration
        - name: plugins-v
          mountPath: /opt/idempiere/plugins
      volumes:
      - name: config-v
        persistentVolumeClaim:
          claimName: idempiere-config-pvc
      - name: plugins-v
        emptyDir: {} # Si no necesitas persistencia crítica en plugins, emptyDir basta
---
apiVersion: v1
kind: Service
metadata:
  name: idempiere-service
spec:
  selector:
    app: idempiere
  ports:
    - name: http
      port: 8080
      targetPort: 8080
    - name: https
      port: 8443
      targetPort: 8443
```

# Crear namespace
```
kubectl create namespace idempiere
```

## Asignar como namespace por defecto
```
kubectl config set-context --current --namespace=idempiere
```

# Desplegar servicio
```
kubectl apply -f pvc.yml
kubectl apply -f deploy_postgres.yml
kubectl apply -f deploy_idempiere.yml
```

# Verificar que se crearon los objetos
```
kubectl get pod,deploy,svc
```