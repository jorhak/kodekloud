```
The xFusionCorp Industries ML team enforces code quality standards using ruff and black for every pull request. The current project located at /root/code/fraud-detection/ is failing both tools. Apply the necessary modifications to ensure it passes the checks for both ruff and black.

The project at /root/code/fraud-detection/ contains a pyproject.toml and sample sources under src/. ruff and black are already installed. From the project directory, run ruff check src/ and black --check src/ to see how they currently fail.

The end state must satisfy the following:

ruff and black are both configured with a line length of 120.

ruff lint rule selection includes E, F, W, and I.

Running ruff check src/ from the project directory exits with status 0.

Running black --check src/ from the project directory exits with status 0.
```
# Pruebas de inicio
#### Prueba de RUFF
```
cd /root/code/fraud-detection
ruff check src/
echo $?
```

Aquí tenemos dos errores:
1. No estamos utilizando un modulo que importamos
2. Una librería esta deprecada.

Para el primer error lo solucionamos quitando la importación y para la librería deprecada la podemos seguir utilizando.
Para corregirlo por medio de los comandos ejecutamos:
```
ruff check src/ --fix
ruff check src/
echo $?
```
Si ejecutamos los comandos podemos evitar modificar el fichero con un editor de texto.
#### Pruebas de BLACK
```
cd /root/code/fraud-detection
black --check src/
echo $?
```

Tenemos un error de que un solo fichero se reformateara y cuatro pemaneceran sin cambios.
Para corregirlo ejecutamos:
```
black src/
black --check src/
echo $?
```

Antes de ejecutar los comandos debemos modificar el fichero pyproject.toml como esta en el paso 3 **Codigo posterior pyproject.toml**.
# 2 Resolucion RUFF
#### Codigo previo src/data/process_data.py
```
import os
import pandas as pd

def load_data(path: str = 'data/raw/transactions.csv') -> pd.DataFrame:
    """Load raw data from a CSV file."""
    return pd.read_csv(path)

def clean_data(df: pd.DataFrame) -> pd.DataFrame:
    """Remove duplicates and null values."""
    df = df.drop_duplicates()
    df = df.dropna()
    return df
```
#### Codigo posteriro src/data/process_data.py
Solo removimos **import os**
```
import pandas as pd

def load_data(path: str = 'data/raw/transactions.csv') -> pd.DataFrame:
    """Load raw data from a CSV file."""
    return pd.read_csv(path)

def clean_data(df: pd.DataFrame) -> pd.DataFrame:
    """Remove duplicates and null values."""
    df = df.drop_duplicates()
    df = df.dropna()
    return df
```
# 3 Resolucion BLACK
#### Codigo previo pyproject.toml
```
[project]
name = "fraud-detection"
version = "0.1.0"

[tool.ruff]
line-length = 88
select = ["E", "F"]

[tool.black]
line-length = 100
```
#### Codigo posterior pyproject.toml
```
[project]
name = "fraud-detection"
version = "0.1.0"

[tool.ruff]
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "W", "I"]

[tool.black]
line-length = 120
```