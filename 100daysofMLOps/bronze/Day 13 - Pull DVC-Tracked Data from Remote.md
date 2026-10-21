```text
A new team member at xFusionCorp Industries has cloned the fraud-detection repository onto a fresh machine. Although the DVC remote is correctly configured to point to the team's SeaweedFS bucket, the dvc pull command is failing. Your task is to diagnose the cause of this failure, correct the configuration as needed, and successfully pull the dataset.


A cloned project exists at /root/code/fraud-detection/ with DVC initialised and the data/raw/transactions.csv.dvc pointer file present, but the dataset itself is missing from disk and from the local DVC cache.

SeaweedFS is already running on the controlplane and the dataset has already been pushed to the dvc-storage bucket — open the SeaweedFS Filer button at the top of the lab and navigate to /buckets/dvc-storage/ to confirm the object is there.

S3 endpoint: http://localhost:8333

Credentials: weedadmin / weedadmin123

Run dvc pull to see it fail, then inspect .dvc/config against the endpoint and credentials above.

Acceptance criteria:

The s3 remote in .dvc/config reaches SeaweedFS with the access key (access_key_id) weedadmin and the secret key (secret_access_key) weedadmin123.

After the pull, data/raw/transactions.csv is present on disk and its content matches the hash recorded in the .dvc pointer.
```
# Ir al directorio
```bash
cd /root/code/fraud-detection/
```
## Ver el error
```bash
dvc pull
```

```text
Collecting                   |1.00 [00:00,  555entry/s]
ERROR: failed to connect to s3 (dvc-storage/files/md5) - Unable to locate credentials
Fetching
ERROR: failed to pull data from the cloud - 1 files failed to download
```
Podemos ver que no tenemos los credenciales para conectarnos a S3, por consecuencia no podemos descargar los ficheros.
# Ver SeaweedFS UI
En el comando anterior vemos que fallo al conectarse con **dvc-storage/files/md5**, vamos a ver si existe esa ruta en la interfaz.
Podemos ver que el directorio existe por lo cual, lo que estaria mal serian nuestros credenciales.
# Configurar credenciales
```bash
vi .dvc/config
```

#### Before
```text
[core]
    remote = s3

['remote "s3"']
    url = s3://dvc-storage
    endpointurl = http://localhost:8333
```
#### After
```bash
[core]
    remote = s3

['remote "s3"']
    url = s3://dvc-storage
    endpointurl =  http://localhost:8333
    access_key_id = weedadmin
    secret_access_key = weedadmin123
```
# Descargar ficheros
```bash
dvc pull
```
# Verificar
```bash
cat data/raw/transactions.csv
```