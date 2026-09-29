```
The xFusionCorp Industries ML team promotes code quality for every commit by utilizing pre-commit. A draft .pre-commit-config.yaml file is located in the git repository at /root/code/fraud-detection/. However, this configuration does not align with the team's standards, resulting in a failure when executing pre-commit run --all-files. Revise the configuration to ensure compliance with the team's requirements.


A git repository already exists at /root/code/fraud-detection/ with .pre-commit-config.yaml and process.py already tracked. pre-commit is installed system-wide. From the project directory, run pre-commit run --all-files to see how the current configuration fails.

The end state must satisfy the following:

the configuration declares these five hooks so that pre-commit run --all-files executes every one of them:
trailing-whitespace, end-of-file-fixer, and check-yaml – All three sourced from the pre-commit/pre-commit-hooks repository, pinned to a current release;
ruff – Sourced from the astral-sh/ruff-pre-commit repository, pinned to a current release;
black – Sourced from the psf/black-pre-commit-mirror repository, pinned to a current release;
every repository entry in the configuration includes a rev: field;
the hooks are registered with git and run cleanly against the tracked files.
Tip: pre-commit autoupdate queries each referenced repository and rewrites the rev: pins to the latest released tag. This is the standard way to discover current versions without looking them up by hand.
```

# 1 Ver el error
```bash
cd fraud-detection
pre-commit run --all-files
```

```bash
cat /root/.cache/pre-commit/pre-commit.log
```

Nos dice que le falta la version y modificamos el fichero **.pre-commit-config.yaml**
### 1.1 Inspeccionar fichero
```bash
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v2.3.0
  hooks:
    - id: trailing-whitespace
    - id: end-of-file-fixer
    - id: check_yaml

- repo: https://github.com/charliermarsh/ruff-pre-commit
  rev: v0.1.0
  hooks:
    - id: ruff-lint

- repo: https://github.com/psf/black-pre-commit-mirror
  hooks:
    - id: black
```
### 1.2 Codigo
```python
def process(x):
    return x + 1
```
Nos vamos al repositorio para ver sus **tags** y actualizamos el fichero
#### Before
```bash
- repo: https://github.com/psf/black-pre-commit-mirror
  hooks:
    - id: black
```
#### After
```bash
- repo: https://github.com/psf/black-pre-commit-mirror
  rev: 26.5.1
  hooks:
    - id: black
```

Volvemos a ejecutar el comando para ver si ya se soluciono el error
```bash
pre-commit run --all-files
```

Nos muestra que debemos ejecutar el comando
```bash
pre-commit autoupdate
pre-commit run --all-files
```
Se soluciono, sin embargo, nos dio un nuevo error de sintaxis nos dice que no se presenta **check_yaml**, nos vamos al repositorio pare ver que valores tiene permitido y lo modificamos
#### Before
```bash
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v2.3.0
  hooks:
    - id: trailing-whitespace
    - id: end-of-file-fixer
    - id: check_yaml
```
#### After
```bash
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v2.3.0
  hooks:
    - id: trailing-whitespace
    - id: end-of-file-fixer
    - id: check-yaml
```

```bash
pre-commit autoupdate
pre-commit run --all-files
```
Volvemos a tener un error **ruff-lint**, se parece al error anterior nos vamos al repositorio y vemos que ese parametro ya no es valido y la url.
#### Before
```bash
- repo: https://github.com/charliermarsh/ruff-pre-commit
  rev: v0.1.0
  hooks:
    - id: ruff-lint
```
#### After
```bash
- repo: https://github.com/astral-sh/ruff-pre-commit
  rev: v0.1.0
  hooks:
    - id: ruff
```

```bash
pre-commit autoupdate
pre-commit run --all-files
```
Volvemos a ejecutar estos comandos, y ahora si todos los test pasan.
# 2 Verificar
El fichero que finalmente con esta configuracion
```bash
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

- repo: https://github.com/astral-sh/ruff-pre-commit
  rev: v0.16.9
  hooks:
    - id: ruff

- repo: https://github.com/psf/black-pre-commit-mirror
  rev: 26.5.1
  hooks:
    - id: black
```
