```
The xFusionCorp Industries ML team utilizes DVC pipelines to ensure the reproducibility of data processing. The fraud-detection project has the processing scripts and raw data in place but does not yet define a pipeline. Define a two-stage DVC pipeline so the data processing runs reproducibly from start to finish with dvc repro.


A project exists at /root/code/fraud-detection/ with DVC initialised. The scripts are at src/data/process_data.py and src/data/split_data.py, and the raw input is at data/raw/transactions.csv. Do not modify the Python files or the input data.

Acceptance criteria:

	- A dvc.yaml defines two stages:
	  1. process_data – runs python3 src/data/process_data.py; depends on 
	     data/raw/transactions.csv and src/data/process_data.py; produces
	     data/processed/clean_transactions.csv.
	
	  2. split_data – runs python3 src/data/split_data.py; depends on 
	     data/processed/clean_transactions.csv (the upstream stage's output, so 
	     DVC chains the stages) and src/data/split_data.py; produces 
	     data/processed/train.csv and data/processed/test.csv.
	
	- The pipeline has been reproduced so both stages execute in order and 
	  dvc.lock is written, and dvc status reports no stale stages.


Use python3 (not python) in the stage commands. Once the pipeline is valid, dvc dag prints the dependency graph showing how the two stages chain together.

```
# Crear DVC PIPELINE
```bash
vi /root/code/fraud-detection/dvc.yaml
```

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
```
#### Ver grafo DAG
```bash
cd /root/code/fraud-detection/
dvc dag
```
# Ejecutar
```bash
dvc repro
```

```text
Running stage 'proces_data':                                         
> python3 src/data/process_data.py
Processed 15 rows
Generating lock file 'dvc.lock'                                      
Updating lock file 'dvc.lock'

Running stage 'split_data':                                          
> python3 src/data/split_data.py
Train: 12 rows, Test: 3 rows
Updating lock file 'dvc.lock'                                        

To track the changes with git, run:

        git add dvc.lock data/processed/.gitignore

To enable auto staging, run:

        dvc config core.autostage true
Use `dvc push` to send your updates to remote storage.
```
# Verificar
```bash
ls -la data/processed
```

```text
-rw-r--r-- 1 root root  537 Oct 21 20:05 clean_transactions.csv
-rw-r--r-- 1 root root  145 Oct 21 20:05 test.csv
-rw-r--r-- 1 root root  441 Oct 21 20:05 train.csv
```

```bash
dvc status
cat dvc.lock
```
