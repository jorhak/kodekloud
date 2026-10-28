```
The xFusionCorp Industries MLOps team versions datasets and models on separate Git branches so it can reproduce and roll between versions cleanly. Tag the current state as `v1.0`, create a `v2-improved` branch built on a newer dataset (which retrains the model), and confirm that switching back restores the original data and model.

  

A project exists at `/root/code/fraud-detection/` with a working DVC pipeline (it processes the data and trains a model) and the baseline `data/raw/transactions.csv` already tracked.

An improved dataset has been pre-staged at `/root/code/fraud-detection/data/raw/transactions_v2.csv` and is visible in the file explorer. Do not delete this file.

Acceptance criteria:

	- On the main branch, the current state is tagged `v1.0`.
	- A branch named `v2-improved` holds the v2 state: the tracked dataset 
	  carries the contents of the v2 file (re-tracked with DVC), the pipeline 
	  has been re-run so `models/model.pkl` is retrained and versioned alongside 
	  the dataset, and the changes are committed.
	- Back on the main branch, the v1 dataset and model are restored on disk, 
	  matching the hashes recorded by the `v1.0` tag.

The DVC extension's **DVC TRACKED** section in the EXPLORER panel reflects the tracked dataset and model for the branch you currently have checked out. To compare the exact hashes recorded on each branch, use `git show <ref>:dvc.lock` or `dvc status`.
```
# Ir a directorio
```bash
cd /root/code/fraud-detection/
git status
dvc status
git branch
git tag
```
# Crear un tag del estado actual
```bash
git tag v1.0
git show
git tag
```
# Crear rama
```bash
git checkout -b v2-improved
git branch
cp data/raw/transactions_v2.csv data/raw/transactions.csv
dvc add data/raw/transactions.csv
```
# Ejecutar
```bash
dvc repro
git status
```
# Seguimiento de nuevos ficheros
```bash
git add data/raw/transactions.csv.dvc dvc.lock
git commit -m "Actualizando dataset y reentrenando modelo v2"
```
# Volver a la rama principal
```bash
git checkout main
dvc checkout
```
# Verificacion
```bash
git show v1.0:dvc.lock
dvc status
```