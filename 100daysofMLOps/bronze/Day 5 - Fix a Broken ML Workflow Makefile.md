```
The xFusionCorp Industries Machine Learning team utilizes a Makefile to streamline essential tasks such as data processing, training, testing, and cleanup. A preliminary Makefile can be found at /root/code/fraud-detection/Makefile, but the execution of make all does not yield successful completion. Ensure that the Makefile is aligned with the team's standards.

A Makefile lives in /root/code/fraud-detection/. Run make all from the project directory to see how it currently fails.

The end state must satisfy the following:

the Makefile declares these six targets and behaviour:

setup – Creates a virtual environment at mlops-venv/ and installs dependencies from requirements.txt;

data – Runs python3 src/data/process_data.py;

train – Runs python3 src/models/train.py;

test – Runs pytest tests/;

clean – Recursively removes every __pycache__ directory, removes .pytest_cache, and clears the contents of models/;

all – Runs setup, data, train, and test in that order;

all six target names are declared as .PHONY so that Make never confuses them with files of the same name;

make all completes without error.

Makefile recipes must be indented with a real tab character, not spaces. Make rejects any recipe that is not tab-indented.
```
# Ver Makefile
```
vi /root/code/fraud-detection/Makefile
```

Modificamos la identacion en el apartado data.
```
# fraud-detection Makefile
.PHONY: help setup data train test clean all

help:
	@echo "Uso: make [target]"
	@echo ""
	@echo "Targets disponibles:"
	@echo "  setup  - Crea el entorno virtual e instala dependencias"
	@echo "  data   - Ejecuta el procesamiento de datos"
	@echo "  train  - Entrena el modelo"
	@echo "  test   - Ejecuta las pruebas unitarias"
	@echo "  clean  - Limpia caches y la carpeta de modelos"
	@echo "  all    - Ejecuta setup, data, train y test en orden"

setup: ## Crea entorno

	python3 -m venv mlops-venv && mlops-venv/bin/pip install -r requirements.txt

data: ## Verifica datos

	python3 src/data/process_data.py

train: ## Entrena modelo

	python3 src/models/train.py

test: ## Ejecucion de test

	pytest tests/

clean: ## Limpiar

	find . -type d -name "__pycache__" -exec rm -rf {} +

	rm -rf .pytest_cache

	rm -rf models/*

all: setup data train test
```
# 2 Verificacion
```
cd fraud-detection/
make help
make all
```