```
The xFusionCorp Industries MLOps team needs every model training run to be reproducible, automatically tracked, and easy to compare so a chosen configuration can be promoted into version control. The fraud-detection pipeline is parameterized by max_depth, currently set shallow enough to underfit. Using DVC experiments, run three tracked experiments over different max_depth values, compare their recorded f1_score on the held-out test set, and promote the best-scoring run so its parameters, metrics, and model become the tracked workspace state.


A project exists at /root/code/fraud-detection/ with a parameterised DVC pipeline already in place. params.yaml declares n_estimators: 100 and max_depth: 4, and the baseline pipeline has been run once. src/models/train.py reads both parameters, trains the model, and evaluates it on the held-out test set, writing the real accuracy and f1_score to metrics.json. Do not modify the Python files.

Acceptance criteria:

- Three DVC experiments have been run, each with a different value for max_depth 
  across a reasonable range (for example 2, 6, and 12); each experiment retrains 
  the model and produces a fresh metrics.json.
- The experiment with the highest f1_score is applied to the workspace, so its 
  max_depth, metrics.json, and models/model.pkl become the tracked state.

The DVC extension's EXPERIMENTS view (open the DVC panel from the Activity Bar) lists every experiment alongside its parameters and metrics, which is a convenient way to compare runs at a glance.
```
# Ir al directorio
```bash
cd /root/code/fraud-detection/
```
# Declarar variables
```bash
vi params.yaml
```

```text
n_estimators: 100
max_depth: 4
```
# Configurar dvc.yaml
```bash
vi dvc.yaml
```

```text
stages:
  process_data:
    cmd: python3 src/data/process_data.py
    deps:
      - data/raw/transactions.csv
      - src/data/process_data.py
    outs:
      - data/processed/clean_transactions.csv

  split_data:
    cmd: python3 src/data/split_data.py
    deps:
      - data/processed/clean_transactions.csv
      - src/data/split_data.py
    outs:
      - data/processed/train.csv
      - data/processed/test.csv

  train:
    cmd: python3 src/models/train.py
    deps:
      - data/processed/train.csv
      - src/models/train.py
    params:
      - n_estimators
      - max_depth
    outs:
      - models/model.pkl
    metrics:
      - metrics.json:
          cache: false
```
# Ver diagrama
```bash
dvc dag
```
# Ver tabla de comparacion
```bash
dvc exp show
```
# Experimento
### Primer experimento
```bash
dvc exp run --set-param max_depth=2
```
### Segundo experimento
```bash
dvc exp run --set-param max_depth=6
```
### Tercer experimento
```bash
dvc exp run --set-param max_depth=12
```
# Verificar
```bash
dvc exp show
```
Debemos seleccionar el mas alto
```text
  ├── fe68aa6 [licit-kind]   04:30 PM       0.75     0.6795   100            12          34832f8da536af18bb25236981b31c04        1d9e91ebeec0d17ccd48207c48305bbe   560443f3cf3752c47513422b98a61a72   20dd83528aa4f1c811acc1999f29b6e0   a8a5e02e0ea8627d58fa9454aa11e2e6   9ce131eb079e5fd3b2f7f15ce16c0fb5  
  ├── 3dc66de [scrap-skin]   04:30 PM      0.745     0.6483   100            6           34832f8da536af18bb25236981b31c04        1d9e91ebeec0d17ccd48207c48305bbe   560443f3cf3752c47513422b98a61a72   20dd83528aa4f1c811acc1999f29b6e0   a8a5e02e0ea8627d58fa9454aa11e2e6   9ce131eb079e5fd3b2f7f15ce16c0fb5  
  └── d7c6ba3 [brisk-divs]   04:30 PM        0.6     0.2453   100            2           34832f8da536af18bb25236981b31c04        1d9e91ebeec0d17ccd48207c48305bbe   560443f3cf3752c47513422b98a61a72   20dd83528aa4f1c811acc1999f29b6e0   a8a5e02e0ea8627d58fa9454aa11e2e6   9ce131eb079e5fd3b2f7f15ce16c0fb5
```

```
dvc exp apply <NOMBRE_O_ID_DEL_EXPERIMENTO>
dvc exp apply licit-kind
dvc metrics diff
```
