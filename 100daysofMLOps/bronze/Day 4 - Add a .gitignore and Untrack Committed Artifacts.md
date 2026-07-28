```
The xFusionCorp Industries fraud-detection repository was committed without a .gitignore file. As a result, Python caches, a trained model file, a virtual environment, notebook checkpoints, and a local secrets file have all been included in version control. Your task is to create a .gitignore file and appropriately stop tracking the artifacts that should not be included in Git.

The Git repository is at /root/code/fraud-detection/. Standard Python / ML artifacts were committed before any .gitignore existed, so ignoring them is not enough — a .gitignore never untracks files Git already tracks.

The end state must satisfy the following:

a .gitignore at the repository root excludes the standard Python / ML artifacts:

Python bytecode caches — __pycache__/ and *.pyc;

virtual environments — venv/;

Jupyter checkpoints — .ipynb_checkpoints/;

trained model files — *.pkl;

local environment files — .env;

those artifacts are removed from Git's index (while remaining on disk) and the cleanup is committed;

the project sources remain tracked: everything under src/fraud_detection/, README.md, and requirements.txt.
```
# 1 Crear fichero .gitignore
```
cd fraud-detection/
vi .gitignore
```

```
__pycache__/
*.pyc
venv/
.ipynb_checkpoints/
*.pkl
.env
```

```
it rm -r --cache src/fraud_detection/__pycache__ models/fraud_model.pkl venv .ipynb_checkpoints .env
git add .gitignore
git commit -m "Add .gitignore"
```
# 2 Verificar
```
git ls-files
ls -a && ls -a models
```