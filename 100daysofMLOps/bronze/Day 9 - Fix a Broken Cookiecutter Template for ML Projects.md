```
The xFusionCorp Industries Machine Learning platform team maintains a Cookiecutter template from which new ML projects are generated. A draft template is available at /root/code/mlops-template/, but it currently does not render properly. Correct the template and use it to generate a new project.


A Cookiecutter template exists at /root/code/mlops-template/. cookiecutter is installed system-wide, and the template is visible in the VS Code explorer. Run cookiecutter /root/code/mlops-template/ to see how it currently fails to render.

The end state must satisfy every one of the following:

The cookiecutter.json declares four variables:
project_name (default my-ml-project)
author (default xFusionCorp)
python_version (default 3.11)
ml_framework with the choices sklearn, pytorch, and tensorflow
The generated requirements.txt logic:
Contains scikit-learn when ml_framework is sklearn
Contains torch when ml_framework is pytorch
Contains tensorflow when ml_framework is tensorflow
The generated README.md content:
Must reference both the project_name and the author from cookiecutter variables.
The template directory structure {{cookiecutter.project_name}}/ must contain:
Files: README.md and requirements.txt
Directories: data/, models/, src/, and tests/
A project generated from the corrected template at /root/code/churn-model/ (with project_name=churn-model and ml_framework=sklearn) contains a requirements.txt listing scikit-learn and a README.md that mentions xFusionCorp.
```
# 1 Crear y activar entorno virtual
```bash
cd mlops-template
python3 -m venv .env
source .env/bin/activate
cd ..
```
# 2 Crear template
```bash
cookiecutter mlops-template
```
Despues de ejecutar el comando nos da un error: que no existe el atributo **Author** y es correcto porque el atributo que tenemos es **author**. Modificamos el README.md
### Before
```python
# {{cookiecutter.project_name}}

Created by {{ cookiecutter.Author }}.
```
### After
```python
# {{cookiecutter.project_name}}

Created by {{ cookiecutter.author }}.
```
Tuvimos un error con la instalacion de los requerimientos por el motivo que contamos con "=" y el mensjase nos sugire que debe ser "\=\=" y tambien que no cerramos el if.
### Before
```python
{% if cookiecutter.ml_framework = 'sklearn' %}
scikit-learn
{% elif cookiecutter.ml_framework = 'pytorch' %}
torch
{% elif cookiecutter.ml_framework = 'tensorflow' %}
tensorflow
```
### After
```python
{% if cookiecutter.ml_framework == 'sklearn' %}
scikit-learn
{% elif cookiecutter.ml_framework == 'pytorch' %}
torch
{% elif cookiecutter.ml_framework == 'tensorflow' %}
tensorflow
{% endif %}
```

Luego nos vuelve a salir un error que no tenemos la variable "ml_framework"
### Before
```json
{
   "project_name": "my-ml-project",
   "author": "xFusionCorp",
   "python_version": "3.11"
}
```
### After
```json
{
   "project_name": "my-ml-project",
   "author": "xFusionCorp",
   "python_version": "3.11",
   "ml_framework": ["sklearn", "tensorflow", "pytorch"]
}
```
# 3 Crear el proyecto especificado
Vamos a colocar **churn-model**. Como nombre del proyecto
```bash
cookiecutter mlops-template
```


