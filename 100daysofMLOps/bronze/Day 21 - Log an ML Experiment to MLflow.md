```
A data scientist at xFusionCorp Industries requires a training run to be recorded in MLflow in order to establish a baseline record on the tracking dashboard. The essential non-MLflow scaffolding has already been implemented in the script located at /root/code/log_experiment.py. Your objective is to complete the script by filling in the TODO blocks with the appropriate MLflow logging calls, ensuring that every aspect of the run is effectively captured by the MLflow tracking server.


1. The MLflow tracking server is already running on port 5000. The MLflow UI 
   button at the top of the lab can be opened to view the dashboard; the Default 
   experiment is present on first load.

2. The script at /root/code/log_experiment.py prepares a params dictionary, fits 
   a trivial sklearn model, and computes a pair of evaluation scores (accuracy 
   and f1) from that model's predictions. Three blocks marked # TODO inside the 
   mlflow.start_run() context are the only edits required.

3. Once the TODOs are completed and the script has been run, the end state must 
   include:

	- A new run in the Default experiment.
	- Every hyperparameter in the params dict (n_estimators=100, max_depth=5, 
	  random_state=42) recorded as a run parameter.
	- Both computed scores (accuracy, f1_score) recorded as run metrics.
	- The sklearn model captured as an MLflow model artefact on the run.

The result can be confirmed in the MLflow UI—once the run is opened, the Parameters, Metrics, and Artifacts panels each show the expected content.
```
# Ingresar a MLflow
Damos click sobre **MLflow UI**->**Default** y vemos que no hay nada por ahora.
Tambien podemos ejecutar el comando:
```bash
curl -I localhost:5000
```
# Configurar fichero
```bash
cd /root/code/
```

```bash
vi log_experiment.py
```
### Before
```python
"""
MLflow experiment logging — three TODO blocks below record a training
run with MLflow.

The model and data in this script are synthetic. A trivial
DummyClassifier stands in for a trained model so that the MLflow
logging calls have a real sklearn estimator to persist, and the
accuracy/F1 scores are computed from its predictions on a small fixed
fixture — not hardcoded. The purpose of the lab is to practise the
MLflow logging API, not to reason about model quality.

The three `# TODO` blocks inside the `mlflow.start_run()` context
are the only edits required.
"""
import numpy as np
import mlflow
import mlflow.sklearn
from sklearn.dummy import DummyClassifier
from sklearn.metrics import accuracy_score, f1_score

mlflow.set_tracking_uri("http://localhost:5000")

# Hyperparameters the run should record as MLflow parameters.
params = {"n_estimators": 100, "max_depth": 5, "random_state": 42}

# Synthetic "trained" model — a DummyClassifier fit on a small set of
# deterministic rows so it has valid internal state for
# mlflow.sklearn.log_model to serialise. No real learning takes place.
X_fit = np.array([[0.0], [1.0], [2.0], [3.0]])
y_fit = np.array([1, 1, 1, 0])
model = DummyClassifier(strategy="most_frequent").fit(X_fit, y_fit)

# Evaluation scores computed from the model's own predictions on the
# fixture above — deterministic and reproducible (accuracy 0.75,
# f1_score ~0.857), not fabricated constants.

preds = model.predict(X_fit)
accuracy = accuracy_score(y_fit, preds)
f1 = f1_score(y_fit, preds)

with mlflow.start_run():

    # TODO 1: log every entry in `params` as an MLflow parameter so that
    # n_estimators, max_depth, and random_state become searchable
    # parameters on this run.


    # TODO 2: log `accuracy` and `f1` as MLflow metrics named
    # "accuracy" and "f1_score" respectively.


    # TODO 3: log the trained `model` as an MLflow sklearn model
    # artefact on this run.
    
    print(f"accuracy={accuracy}, f1_score={f1}")
```
### After
```python
"""
MLflow experiment logging — three TODO blocks below record a training
run with MLflow.

The model and data in this script are synthetic. A trivial
DummyClassifier stands in for a trained model so that the MLflow
logging calls have a real sklearn estimator to persist, and the
accuracy/F1 scores are computed from its predictions on a small fixed
fixture — not hardcoded. The purpose of the lab is to practise the
MLflow logging API, not to reason about model quality.

The three `# TODO` blocks inside the `mlflow.start_run()` context
are the only edits required.
"""
import numpy as np
import mlflow
import mlflow.sklearn
from sklearn.dummy import DummyClassifier
from sklearn.metrics import accuracy_score, f1_score

mlflow.set_tracking_uri("http://localhost:5000")

# Hyperparameters the run should record as MLflow parameters.
params = {"n_estimators": 100, "max_depth": 5, "random_state": 42}

# Synthetic "trained" model — a DummyClassifier fit on a small set of
# deterministic rows so it has valid internal state for
# mlflow.sklearn.log_model to serialise. No real learning takes place.
X_fit = np.array([[0.0], [1.0], [2.0], [3.0]])
y_fit = np.array([1, 1, 1, 0])
model = DummyClassifier(strategy="most_frequent").fit(X_fit, y_fit)

# Evaluation scores computed from the model's own predictions on the
# fixture above — deterministic and reproducible (accuracy 0.75,
# f1_score ~0.857), not fabricated constants.

preds = model.predict(X_fit)
accuracy = accuracy_score(y_fit, preds)
f1 = f1_score(y_fit, preds)

with mlflow.start_run():

    # TODO 1: log every entry in `params` as an MLflow parameter so that
    # n_estimators, max_depth, and random_state become searchable
    # parameters on this run.
    mlflow.log_params(params)


    # TODO 2: log `accuracy` and `f1` as MLflow metrics named
    # "accuracy" and "f1_score" respectively.
    mlflow.log_metric("accuracy",accuracy)
    mlflow.log_metric("f1_score",f1)

    # TODO 3: log the trained `model` as an MLflow sklearn model
    # artefact on this run.
    mlflow.sklearn.log_model(model, name="model")
    
    print(f"accuracy={accuracy}, f1_score={f1}")
```
## Ejecutar
```bash
python3 log_experiment.py
```
# Verificar 
Nos volvemos al navegador y una ves en **Default** damos click sobre **Training runs** que esta a la izquierda, luego damos click sobre el mas reciente porque le genera un nombre random.