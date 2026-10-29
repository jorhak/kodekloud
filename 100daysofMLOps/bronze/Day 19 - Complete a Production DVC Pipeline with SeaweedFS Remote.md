```
Complete the xFusionCorp Industries fraud-detection production DVC pipeline. Three stages are already wired in dvc.yaml, two remain, and the pipeline must finish as a reproducible, SeaweedFS-backed, v1.0-tagged release.


A project exists at /root/code/ml-pipeline/ with Git and DVC initialised. The params.yaml is in place and the .dvc/config is pre-configured to push to the SeaweedFS bucket dvc-storage at http://localhost:8333.

The ingest, validate, and preprocess stages are already declared in dvc.yaml, but one of them is misconfigured and prevents dvc repro from completing — run dvc repro to see it fail. The two scripts for the remaining stages are pre-staged at /root/code/ml-pipeline/scripts-staging/train.py and scripts-staging/evaluate.py, and belong in scripts/.

Acceptance criteria:

- The misconfigured existing stage is corrected so dvc repro can complete.
- Two further stages are declared in dvc.yaml:
  - train – Depends on the preprocessed dataset and scripts/train.py; reads 
    n_estimators, max_depth, test_size, and random_seed from params.yaml; 
    outputs models/model.pkl and data/processed/test_split.csv; declares 
    metrics.json as a DVC metric with cache: false.
  - evaluate – Depends on models/model.pkl, data/processed/test_split.csv, and 
    scripts/evaluate.py; outputs reports/evaluation.json declared with cache: 
    false.
- The full pipeline has been reproduced, the cache pushed to the SeaweedFS 
  remote, and the current state tagged v1.0.
- Every change is committed to Git so the release is fully captured.

Open the SeaweedFS Filer button at the top of the lab and navigate to /buckets/dvc-storage/ to confirm that the bucket holds the pushed artefacts under the files/md5/... layout.
```
# Ir al directorio
```bash
cd /root/code/ml-pipeline/
```
### Ver el diagrama
```bash
dvc dag
```
### Ver configuracion
```bash
vi .dvc/config
```

```text
[core]
    remote = s3

['remote "s3"']
    url = s3://dvc-storage
    endpointurl = http://localhost:8333
    access_key_id = weedadmin
    secret_access_key = weedadmin123
```
# Inspeccionar el error
```bash
dvc repro
```

```text
Running stage 'ingest':                              
> python3 scripts/ingest.py
Data ingested successfully: 20 rows, 5 columns
Generating lock file 'dvc.lock'                      
Updating lock file 'dvc.lock'

Running stage 'validate':                            
> python3 scripts/validate.py
Validation: 20 rows, valid=True
Updating lock file 'dvc.lock'                        

Running stage 'preprocess':                          
> python3 scripts/preprocess.py
Preprocessed: 20 clean rows
ERROR: failed to reproduce 'preprocess': output 'data/processed/cleaned.csv' does not exist
```
#### Verificamos ese fichero
```bash
ls data/processed/
```

```text
clean.csv
```
Podemos ver que los nombres no coinciden vamos a modificar el fichero y volvemos a ejecutar.
### Ver dvc.yaml
```bash
vi dvc.yaml
```
#### Before
```text
stages:
  ingest:
    cmd: python3 scripts/ingest.py
    deps:
      - scripts/ingest.py
      - data/raw/data.csv

  validate:
    cmd: python3 scripts/validate.py
    deps:
      - data/raw/data.csv
      - scripts/validate.py
    outs:
      - reports/validation.json:
          cache: false

  preprocess:
    cmd: python3 scripts/preprocess.py
    deps:
      - data/raw/data.csv
      - scripts/preprocess.py
    outs:
      - data/processed/cleaned.csv
```
#### After
```text
stages:
  ingest:
    cmd: python3 scripts/ingest.py
    deps:
      - scripts/ingest.py
      - data/raw/data.csv

  validate:
    cmd: python3 scripts/validate.py
    deps:
      - data/raw/data.csv
      - scripts/validate.py
    outs:
      - reports/validation.json:
          cache: false

  preprocess:
    cmd: python3 scripts/preprocess.py
    deps:
      - data/raw/data.csv
      - scripts/preprocess.py
    outs:
      - data/processed/clean.csv
```
### Volver a ejecutar
```bash
dvc repro
```
# Copiar fichero train, evaluate
```bash
cp scripts-staging/train.py scripts/
cp scripts-staging/evaluate.py scripts/
ls scripts/
```
# Agregar train, evaluate a dvc
Agregamos al final del fichero
```bash
vi dvc.yaml
```

```yml
  train:
    cmd: python3 scripts/train.py
    deps:
      - data/raw/data.csv
      - scripts/train.py
      - data/processed/clean.csv
    outs:
      - models/model.pkl
      - data/processed/test_split.csv
    params:
      - n_estimators
      - max_depth
      - test_size
      - random_seed
    metrics:
      - metrics.json:
          cache: false

  evaluate:
    cmd: python3 scripts/evaluate.py
    deps:
      - models/model.pkl
      - data/processed/test_split.csv
      - scripts/evaluate.py
    outs:
      - reports/evaluation.json:
          cache: false
```
### Ver diagrama
```bash
dvc dag
```
### Ejecutar
```bash
dvc repro
```
# Subir artefactos a repositorio remoto SeaweedFS
```bash
dvc push
```
# Commit a ficheros y directorios
```bash
git add .
git commit -m "Pipeline ejecutado"
```
# Crear tag
```bash
git tag v1.0
```
# Verificar
```bash
git show
git tag -l
dvc status
dvc metrics show
```
En la parte superio presionamos el boton **SeaweedFS Filer**.
```bash
AWS_ACCESS_KEY_ID=weedadmin AWS_SECRET_ACCESS_KEY=weedadmin123 \
aws --endpoint-url=http://localhost:8333 s3 ls s3://dvc-storage/ --recursive | head
```
Este comando lo podemos encontrar en la documentacion de [SeaweedFS Filer](https://github.com/seaweedfs/seaweedfs).