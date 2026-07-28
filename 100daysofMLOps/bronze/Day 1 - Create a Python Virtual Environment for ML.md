```
The xFusionCorp Industries data science team requires a standardized Python environment for their new machine learning project. Set up a virtual environment on the controlplane host that includes all necessary ML libraries.

Create a Python virtual environment named ml-env under /root/code/ using python3 -m venv.

Activate the environment and install the following packages: numpy, pandas, scikit-learn, and matplotlib.

Generate a requirements.txt file using pip freeze and save it at /root/code/requirements.txt.
```
# 1 Crear enviroment
```
python3 -m venv ml-env
```
#### Activar enviroment
```
source ml-env/bin/activate
```
#### Instalar packages
```
pip install numpy pandas scikit-learn matplotlib
```
#### Generar requirements.txt
```
pip freeze > requirements.txt
```
