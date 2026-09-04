```
The xFusionCorp Industries deployment team requires the fraud-detection module to be validated through unit tests and to be packaged as an installable Python distribution. The module's source code and a draft pyproject.toml file can be found at /root/code/fraud-detection/. Your task is to create unit tests for the module, rectify the packaging configuration, and build a compliant wheel.


The project at /root/code/fraud-detection/ contains the module source under src/fraud_detection/ — a predict() function that flags a transaction as fraud when its amount (the first feature value) exceeds 100. The source is complete; you do not need to modify it. pytest and build are already installed. Use python3 rather than python.

The end state must satisfy the following:

- Unit tests: tests/test_predict.py contains at least two tests that import predict from fraud_detection and assert on its output — one fraudulent row (amount > 100, expect 1) and one legitimate row (amount <= 100, expect 0); pytest run from the project directory passes.

- Packaging configuration: the corrected pyproject.toml satisfies every one of the following:
    - a [build-system] section with requires = ["setuptools>=61.0", "wheel"] and build-backend = "setuptools.build_meta";
    - name is fraud_detection;
    - version is 0.1.0;
    - requires-python is >=3.10;
    - dependencies is ["scikit-learn", "pandas", "numpy"];
    - pytest can import the package from src/ — declare [tool.pytest.ini_options] with pythonpath = ["src"].

- Built artifact: building the package produces a wheel named fraud_detection-0.1.0-*.whl under dist/.
```
# 1 Inspeccionar el proyecto
## 1.1 Revisar el contenido del proyecto
Hacemos esto para poder enter su funcionamiento y empezar a crear el test.
**predict.py**
```python
"""Fraud prediction inference module."""
from typing import Iterable, List

def predict(data: Iterable[Iterable[float]]) -> List[int]:
    """Return a fraud label (0 or 1) for each input feature row.
    This is a placeholder implementation that flags any transaction whose
    first feature value exceeds 100 as fraudulent. The real model loader
    is wired in elsewhere in the deployment pipeline.
    """
    return [1 if row[0] > 100 else 0 for row in data]
```
**/\_\_init/\_\_.py**
```python
"""Fraud detection model package."""

__version__ = "0.1.0"

from .predict import predict

__all__ = ["predict"]
```
# 1.1 Crear test
```bash
vi /root/code/fraud-detection/tests/test_predict.py
```

```python
from fraud_detection import predict

def test_predict_legitimate_transaction():
    # Monto <= 100 debe retornar 0
    data = [[50.0, 1.2], [100.0, 0.5]]
    assert predict(data) == [0, 0]
def test_predict_fraudulent_transaction():
    # Monto > 100 debe retornar 1
    data = [[100.1, 2.0], [250.0, 0.0]]
    assert predict(data) == [1, 1]
```
# 2 Configurar pyproject.toml
```bash
vi /root/code/fraud-detection/pyproject.toml
```

```python
[build-system]
requires = ["setuptools>=61.0","wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "fraud_detection"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    "scikit-learn",
    "pandas",
    "numpy",
]

[tool.pytest.ini_options]
pythonpath = ["src"]
```
# 3 Ejecutar test
```bash
cd /root/code/fraud-detection
pytest
```
# 4 Compilar artefacto
```
python3 -m build
```
Esto va generar un ficher en **dist/**.
# 5 Verificar
```
ls dist/
```