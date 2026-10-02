```
A teammate has added the transactions dataset to the xFusionCorp Industries fraud-detection repository. However, it was committed directly to Git rather than being tracked with DVC. Your task is to align the repository with the team standard, ensuring that every dataset under the `data/` directory is tracked by DVC instead of Git.

  

A project exists at `/root/code/fraud-detection/` with DVC already initialised. The dataset `data/raw/transactions.csv` is currently tracked by Git, and the team standard requires DVC to own it instead.

Acceptance criteria:

- Git no longer tracks the dataset, but the file remains on disk.
- The dataset is tracked by DVC instead: a `.dvc` pointer file exists and `data/raw/.gitignore` excludes the dataset itself.
- The new `.dvc` pointer and `.gitignore` are recorded in a Git commit with the message `Track transactions dataset with DVC`.

> Once tracking is moved to DVC, the **DVC TRACKED** section in the EXPLORER panel will list the dataset, confirming the extension recognises it as a DVC-managed file.
```
# 1. Ir al directorio
```bash
cd /root/code/fraud-detection
```
#### 1.1 Ver los logs
```bash
git log -n3
```
# 2. Agregar data/raw/transactions.csv
```bash
dvc add data/raw/transactions.csv
```

```
dvc add data/raw/transactions.csv
Adding...                                                            
ERROR:  output 'data/raw/transactions.csv' is already tracked by SCM (e.g. Git).
    You can remove it from Git, then add to DVC.
        To stop tracking from Git:
            git rm -r --cached 'data/raw/transactions.csv'
            git commit -m "stop tracking data/raw/transactions.csv" 
```
Ejecutamos los comandos que nos suguiere:
```bash
git rm -r --cached 'data/raw/transactions.csv'
git commit -m "stop tracking data/raw/transactions.csv"
```
Ahora si volvemos a ejecutar:
```bash
dvc add data/raw/transactions.csv
```

```bash
100% Adding...|█████████████████████████████|1/1 [00:00, 37.34file/s]

To track the changes with git, run:

        git add data/raw/transactions.csv.dvc data/raw/.gitignore

To enable auto staging, run:

        dvc config core.autostage true
```
Vamos a ejecutar los comandos que nos suguiere:
```bash
git add data/raw/transactions.csv.dvc data/raw/.gitignore
git commit -m "Track transactions dataset with DVC"
```
# 3. Verificar 
```bash
git log -n3
```
# data/raw/transactions.csv
```csv
transaction_id,amount,merchant,category,is_fraud
1001,25.50,StoreA,groceries,0
1002,1250.00,OnlineShopB,electronics,1
1003,45.00,RestaurantC,dining,0
1004,890.00,StoreD,clothing,0
1005,3200.00,OnlineShopE,electronics,1
1006,12.99,StoreF,groceries,0
1007,567.00,StoreG,clothing,0
1008,2100.00,OnlineShopH,electronics,1
1009,33.50,RestaurantI,dining,0
1010,78.00,StoreJ,groceries,0
```