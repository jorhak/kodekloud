```
A teammate has configured a JupyterLab server for the xFusionCorp Industries data science team; however, the server is not functioning as expected. Inspect the configuration, diagnose any issues, and start the server.

JupyterLab is already installed in the virtual environment at /root/code/ml-env/. The team's configuration file is at /root/code/jupyter_lab_config.py and is visible in the file explorer. Start the server with the config (e.g. /root/code/ml-env/bin/jupyter lab --config /root/code/jupyter_lab_config.py) and observe how it comes up so you can see what is misconfigured.

The end state must satisfy the following:

the running server listens on port 8888;

it binds on 0.0.0.0;

the notebook root directory is /root/notebooks/, and that directory exists on disk.

With the configuration corrected and JupyterLab running, the Jupyter UI button at the top of the lab opens the notebook interface.
```
# 1 Habilitar enviroment
```
source ml-env/bin/activate
```
#### Modificar fichero jupyter_lab_config.py
```
vi jupyter_lab_config.py
```

```
c.IdentityProvider.token = ''
c.ServerApp.disable_check_xsrf = True
c.ServerApp.root_dir = '/root/notebooks'
c.ServerApp.port = 8888
c.ServerApp.ip = '0.0.0.0'
```
Crea directorio
```
mkdir /root/notebooks
```
#### Levantar servidor
Antes debemos instalar notebook
```
pip install notebook
```

```
jupyter lab --config jupyter_lab_config.py --allow-root
```