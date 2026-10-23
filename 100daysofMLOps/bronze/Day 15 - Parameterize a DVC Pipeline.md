```
The xFusionCorp Industries ML team manages model hyperparameters using params.yaml, enabling experiments to be conducted without altering the code. In the fraud-detection project, the train stage retrieves the n_estimators parameter from params.yaml, but this parameter is not declared to DVC, which means that changing its value does not initiate retraining. Integrate the parameter into the pipeline and illustrate the concept of parameter-driven reproducibility.


A project exists at /root/code/fraud-detection/ with a three-stage DVC pipeline (process_data, split_data, train) and a params.yaml declaring n_estimators: 100. src/models/train.py already reads n_estimators from params.yaml. Do not modify the Python files.

The train stage in dvc.yaml currently has no params: section, so DVC does not track n_estimators — changing it would not re-run the stage.

Acceptance criteria:

- The train stage in dvc.yaml lists n_estimators under a params: section, and 
  the pipeline has been reproduced.
- Parameter-driven retraining is demonstrated: with n_estimators changed to a 
  different value (for example 200), re-running the pipeline re-executes only 
  the train stage, records the new value in dvc.lock, and regenerates 
  models/model.pkl.

dvc params diff reports changes to the tracked parameter values across Git commits, which is useful when comparing experiments.
```
# Ir al directorio
```bash
cd /root/code/fraud-detection/
```
## Ver params.yaml
```bash
vi params.yaml
```

```text
n_estimators: 100
```
### Ejecutar
Antes de modificar vamos a ejecutar el pipeline
```bash
dvc repro
```
## Ver dvc.yaml
```bash
vi dvc.yaml
```
### Before
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
    outs:
      - models/model.pkl
```
### After
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
    outs:
      - models/model.pkl
```
# Ver diagrama
```
dvc dag
```
# Cambiar valor
```bash
echo "n_estimators: 250" > params.yaml
```
## Volver a ejecutar solo train
```bash
dvc repro
```
# Verificar
```bash
dvc params diff
```