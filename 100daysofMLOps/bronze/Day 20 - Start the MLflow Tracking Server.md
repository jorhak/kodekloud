```
The xFusionCorp Industries ML team is in the process of adopting MLflow for their experiment tracking. Your task is to set up a local MLflow tracking server on the ML pipeline workstation, enabling the team to log experiments from their training code.


MLflow is pre-installed on the controlplane. Launch the tracking server in the background and choose the flags that satisfy every end-state requirement below.

1. The server listens on port 5000 and is reachable on all network interfaces, 
   not only localhost.

2. The backend store is a SQLite database at /root/code/mlflow-
   backend/mlflow.db. Create any parent directory first — MLflow aborts at 
   startup if the backend directory is missing.

3. The artifact root is /root/code/mlflow-artifacts/.

4. The MLflow UI button at the top of the lab routes through the lab proxy, 
   which reaches the server with a non-localhost host header and a different 
   origin. Launch the server so it accepts any host header and any origin; 
   otherwise the button returns a 403 or CORS error.

5. The server process persists in the background so it survives terminal closure.

Once the server is running, the Default experiment can be viewed from the MLflow UI button. The experiment is empty.
```
# Crear directorios
```bash
mkdir -p /root/code/mlflow-backend /root/code/mlflow-artifacts
```
# Inicialiazar servidor
```bash
mlflow server \
  --backend-store-uri sqlite:////root/code/mlflow-backend/mlflow.db \
  --default-artifact-root /root/code/mlflow-artifacts/ \
  --host 0.0.0.0 \
  --port 5000 \
  --allowed-hosts '*' \
  --cors-allowed-origins '*' > mlflow.log 2>&1 &
```
# Verificacion
```bash
curl -s http://localhost:5000 | head
# o
curl -I http://localhost:5000
```
Finalmente podes dar click sobre el boton **MLflow UI**.
[MLflow](https://mlflow.org/)
