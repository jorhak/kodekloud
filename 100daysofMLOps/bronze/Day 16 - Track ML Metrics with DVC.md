```
After training a model, the xFusionCorp Industries ML team requires DVC to surface model metrics through dvc metrics show. Although the fraud-detection pipeline successfully trains a model and generates a metrics.json file, DVC currently does not recognize this file as a metric. Ensure that the metrics.json file is properly configured to be recognized by DVC.


A project exists at /root/code/fraud-detection/ with a three-stage DVC pipeline (process_data, split_data, train). The train stage runs src/models/train.py, which writes the model to models/model.pkl and metrics to metrics.json. Do not modify the Python files.

Acceptance criteria:

- The train stage in dvc.yaml declares metrics.json as a DVC metric output 
  rather than a regular file output, with cache: false so the JSON lives in Git 
  for diff history rather than in the DVC cache.

- The pipeline has been reproduced so the metric registration takes effect, and 
  dvc metrics show reports the accuracy and f1_score values from metrics.json.

Tip: once the metric is registered, dvc metrics diff compares its values across Git commits, which is useful when iterating on the model.
```
# Ir a directorio
```bash
cd /root/code/fraud-detection && ls
```
## Ver dvc.yaml
```bash
vi dvc.yaml
```
### Before
```yml
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
    outs:
      - models/model.pkl
      - metrics.json
```
### After
```yml
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
    outs:
      - models/model.pkl
    metrics:
      - metrics.json:
          cache: false
```
### Ejecutar
```bash
dvc repro
```
# Verificar
```bash
dvc metrics show
dvc metrics diff
```