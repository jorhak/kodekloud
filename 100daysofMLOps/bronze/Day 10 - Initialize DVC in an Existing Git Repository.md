```
The xFusionCorp Industries ML team is implementing DVC to ensure that datasets and model files are versioned independently from the codebase. Initialize DVC within the existing Git repository located at `/root/code/fraud-detection/` and record this initialization in Git.

  

A Git repository already exists at `/root/code/fraud-detection/` with an initial commit.

Acceptance criteria:

- DVC is initialised inside that repository, so the standard `.dvc/` control directory and `.dvcignore` file exist alongside the existing Git working tree.
- Every file DVC produces during initialisation is recorded in a new Git commit with the message `Initialize DVC`.

> Once initialisation is complete, the **DVC** extension will detect the new `.dvc/` directory and surface the **DVC TRACKED** section in the EXPLORER panel together with a `DVC` indicator in the bottom status bar.
```
# 1.Ir al directorio
```bash
cd /root/code/fraud-detection
```
### 1.1 Ver el estado de git
```bash
git status
git log -n3
```
# 2. Inicializar DVC
```bash
dvc init
```
[DVC, user guide, analytics](https://doc.dvc.org/user-guide/analytics)
[Documentacion](https://dvc.org/doc)
[Ayuda](https://dvc.org/chat)
[Github](https://github.com/treeverse/dvc)
# 3. Ver que directorios y ficheros son nuevos
```bash
git status
```
### 3.1 Moverlos al area de trabajo
```bash
git add .
```
### 3.2 Agregarlos al area de stage
```bash
git commit -m "Initialize DVC"
```
# 4. Verificar
```bash
git log -n3
```

