```
A data scientist at xFusionCorp Industries has completed ten runs in the `fraud-detection` MLflow experiment. Your objective is to triage these runs within the **MLflow UI**. Begin by utilizing the search functionality to narrow down the results. Subsequently, compare the candidates side by side and identify the single best candidate to tag as shortlisted. Additionally, flag any runs that are clearly underperforming for removal.

  

1. The MLflow tracking server is already running on port `5000`, and the `fraud-detection` experiment has been pre-populated with ten runs (each carrying `n_estimators`, `max_depth`, `accuracy`, and `f1_score`). The runs can be viewed via the **MLflow UI** button → `fraud-detection` experiment.
    
2. Work through the MLflow UI's **Search** bar (to filter runs with the `metrics.*` query syntax) and its **Compare** view (to inspect the contenders side by side) to reach the triage end state below. The end state is what is tested—the path taken through the UI is not.
    
    - **Shortlist the best candidate.** Among all runs where `metrics.f1_score > 0.85`, the single run with the highest `f1_score` must carry a run-level tag: key `review-status`, value `shortlisted`.
        
    - **Reject the under-performers.** Every run where `metrics.f1_score < 0.75` must carry a run-level tag: key `review-status`, value `rejected`.
        
3. The other runs (those in the 0.75 ≤ f1 ≤ 0.85 band, and the second-best shortlisting candidate) must carry no `review-status` tag at all.
```
# Abrir MLflow
Damos click en la parte superior derecha sobre el boton **MLflow UI**->**fraud-detection**.
Ahora nos dirigimos en la parte inferior izquierda y buscamos **Evaluations runs** damos clic sobre el.
# Buscar
## >0.85
Vamos a buscar quien cumple con el requerimiento **metrics.f1_score>0.85** en la barra de filtros.
El que cumple con este requerimiento son:

| Run Name            | f1_score | accuracy |
| ------------------- | -------- | -------- |
| mercurial-bat-572   | 0.95     | 0.94     |
| worried-dog-860     | 0.93     | 0.92     |
| enchanting-tern-103 | 0.91     | 0.89     |
| persistent-doe-264  | 0.88     | 0.87     |
| silent-wasp-99      | 0.86     | 0.85     |
Vamos a selecionar a **mercurial-bat-572** porque es el que tiene un valor mayor a los demas, damos click en el icono _Go to the run_, nos va dirigir a la pestana **Traces** nos vamos a dirigir a **Overview**, en la parte inferior buscamos _Tags_, click en **Add tags**, se nos va abrir una ventana y vamos agregar en Key: **review-status** y en Value: **shortlisted**, para terminar click en **Save tags**.
## <0.75
Volvemos a buscar quien cumple con el requerimiento **metrics.f1_score<0.75** en la barra de filtros. Los que cumplen con este requerimiento son:

| Run Name           | f1_score | accuracy |
| ------------------ | -------- | -------- |
| oderly-toad-825    | 0.72     | 0.7      |
| dazzling-sheep-407 | 0.7      | 0.68     |
Vamos a selecionar el que tenga un valor mas alto, damos click en el icono de _Go to the run_, nos va ha dirigir a la pestana **Traces**, nos vamos a dirigir a **Overview**, en la parte inferior buscamos _Tags_, click en **Add tags**, se nos va abrir una ventana y vamos a agregar en Key: **review-status** y en Value: **rejected**, click en **Save tags**.
Repetimos todo lo anterior en cada experimento.
# Verificar 
Borramos lo que tenga nuestro filtro, y damos click en **Group by** y marcamos **review-status**.

Tambien podemos ejecutar este comando en la terminal
```python
python3 -c "
from mlflow import MlflowClient
c = MlflowClient('http://localhost:5000')
exp = c.get_experiment_by_name('fraud-detection')
for r in c.search_runs([exp.experiment_id]):
    tag = r.data.tags.get('review-status', '-')
    f1 = r.data.metrics.get('f1_score')
    print(f'{r.info.run_id[:8]}  f1={f1:.2f}  review-status={tag}')
"
```

```text
85c962a8  f1=0.70  review-status=rejected
d13ce3c2  f1=0.95  review-status=shortlisted
cd17464a  f1=0.93  review-status=-
c94609fc  f1=0.91  review-status=-
9695e1fa  f1=0.88  review-status=-
36b17bee  f1=0.86  review-status=-
8931d0ec  f1=0.84  review-status=-
ac9dced2  f1=0.81  review-status=-
416db230  f1=0.78  review-status=-
2739327e  f1=0.72  review-status=rejected
```
# Script
Tambien podriamos usar este script que realizara la tarea
#### Agregar tags
```python
import mlflow
from mlflow.tracking import MlflowClient

# 1. Conectar al servidor local de MLflow
mlflow.set_tracking_uri("http://127.0.0.1:5000")
client = MlflowClient()

# 2. Obtener el experimento 'fraud-detection'
experiment = client.get_experiment_by_name("fraud-detection")
if experiment is None:
    raise ValueError("No se encontró el experimento 'fraud-detection'.")

# 3. Buscar todas las ejecuciones (runs) del experimento
runs = client.search_runs(experiment_ids=[experiment.experiment_id])

# Limpiar etiquetas previas de 'review-status' por seguridad
for run in runs:
    if "review-status" in run.data.tags:
        client.delete_tag(run.info.run_id, "review-status")

# 4. Identificar el único mejor candidato (metrics.f1_score > 0.85)
top_candidates = [
    r for r in runs if r.data.metrics.get("f1_score", 0) > 0.85
]

if top_candidates:
    best_run = max(top_candidates, key=lambda r: r.data.metrics["f1_score"])
    client.set_tag(best_run.info.run_id, "review-status", "shortlisted")
    print(f"[SHORTLISTED] Run ID: {best_run.info.run_id} | f1_score: {best_run.data.metrics['f1_score']:.4f}")

# 5. Etiquetar como 'rejected' todos los runs con metrics.f1_score < 0.75
for run in runs:
    f1 = run.data.metrics.get("f1_score")
    if f1 is not None and f1 < 0.75:
        client.set_tag(run.info.run_id, "review-status", "rejected")
        print(f"[REJECTED]    Run ID: {run.info.run_id} | f1_score: {f1:.4f}")

print("\n¡Triaje completado con éxito!")
```
#### Verificar tag
```python
import mlflow
from mlflow.tracking import MlflowClient

mlflow.set_tracking_uri("http://127.0.0.1:5000")
client = MlflowClient()

experiment = client.get_experiment_by_name("fraud-detection")
runs = client.search_runs(
    experiment_ids=[experiment.experiment_id],
    order_by=["metrics.f1_score DESC"]
)

print(f"{'RUN NAME':<20} | {'F1_SCORE':<10} | {'REVIEW-STATUS TAG'}")
print("-" * 55)

for i, run in enumerate(runs):
    f1 = run.data.metrics.get("f1_score", 0.0)
    tag = run.data.tags.get("review-status", "(sin etiqueta)")
    
    # Si no es el #1 (i == 0 con f1 > 0.85) ni es < 0.75, nos aseguramos de borrar la etiqueta si existiera
    if not (i == 0 and f1 > 0.85) and not (f1 < 0.75):
        if "review-status" in run.data.tags:
            client.delete_tag(run.info.run_id, "review-status")
            tag = "(etiqueta eliminada)"

    print(f"{run.info.run_name:<20} | {f1:<10.4f} | {tag}")

```
