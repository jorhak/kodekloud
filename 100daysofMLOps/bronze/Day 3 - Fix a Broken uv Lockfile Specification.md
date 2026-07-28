```
The xFusionCorp Industries ML team utilizes uv and lockfiles to maintain consistent Python dependencies across different machines. A teammate has submitted a requirements.in specification that does not adhere to the team's standards. Correct the specification and compile it into a pinned lockfile.

A high-level dependency specification exists at /root/code/fraud-detection/requirements.in, but it does not match the team's standards. uv is already installed.

The end state must satisfy the following:

the corrected requirements.in lists exactly these four top-level packages: scikit-learn, mlflow, pandas, and numpy, with any version constraint being one uv can satisfy against PyPI (bare package names are fine — uv pins exact versions when it compiles the lockfile);

a pinned lockfile requirements.txt is compiled from the corrected specification, pinning each of the four top-level packages to an exact version using == and including the transitive dependencies that uv resolved.
```
# 1 Modificar fichero requirements.in
```
scikit-learn
mlflow
pandas
numpy
```
# 2 Compilar lockfile
```
cd /root/code/fraud-detection
uv pip compile requirements.in -o requirements.txt
```