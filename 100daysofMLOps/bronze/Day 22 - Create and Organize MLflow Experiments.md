```
You are tasked with onboarding two new ML projects for the xFusionCorp Industries ML platform team. It is essential that each project is organized under its own MLflow experiment, rather than sharing the Default experiment. Proceed to register both experiments using the MLflow UI, ensuring that they are correctly tagged with the owning team.


1. The MLflow tracking server is already running on port 5000. The MLflow UI 
   button at the top of the lab can be opened to view the dashboard. One seeded 
   experiment (legacy-models) is listed alongside the platform-created Default—
   both act as reference material and must not be modified.

2. Using the MLflow UI, register two new experiments with the experiment-level 
   metadata below. The task is complete when both records satisfy every bullet.

	- fraud-detection

		- Experiment-level description is a non-empty string describing the 
		  project (any phrasing).
		- Experiment-level tag: key team, value ml-platform.

	- churn-prediction

		- Experiment-level tag: key team, value analytics.

The result can be confirmed in the MLflow UI: both new experiments appear in the left-hand list, with the description and tags visible on each experiment's page.
```
# Crear fraud-detection
Damos click sobre el boton **MLflow UI**->**Experiments**->**Create** le asignamos el nombre **fraud-detection**, presionamos _Create_.
Vamos a ingresar al experimento recien creado, en la parte superior derecha damos click sobre el icono de los tres puntos, damos click en _Edit experiment_,
colocamos la descripcion luego presionamos **Save**.
Para luego en la parte superior izquierda dar click sobre **fraud-detection** y lo buscamos en la lista de experimentos, para agregale _tags_, damos click en **Add tags**, agregamos el tag y en la parte de key cuando ingresamos la _key_, se va ha desplegar "Add tag \<key\>" damos click sobre el para agregar la nueva _key_, luego en la derecha damos click sobre el boton de **MAS**, y finalmente click en **Save tags**.
# Crear churn-prediction
Realizamos estas mismas acciones, solo que sin la descripcion.
# Verifiacion
### Version 1
```python
from mlflow import MlflowClient

client = MlflowClient('http://localhost:5000')
e1 = client.search_experiments(filter_string="name = 'fraud-detection'")
e2 = client.search_experiments(filter_string="name = 'churn-prediction'")
print(e1[0].name, e1[0].tags)
print(e2[0].name, e2[0].tags)
```

### Version 2
```python
from mlflow import MlflowClient

client = MlflowClient('http://localhost:5000')
experiments = ['fraud-detection','churn-prediction']
for e in experiments:
   e1 = client.search_experiments(filter_string=f"name = '{e}'")
   print(f"Nombre del experimento: {e1[0].name} \n Tags {e1[0].tags}")
```
