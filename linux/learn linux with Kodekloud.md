# Group Creation and User Assignment
## Crear grupo
```
sudo groupadd nautilus_noc
```

## Crear usuario y asignar grupo
```
sudo useradd -g nautilus_noc -c "Hola Sonya" -d /home/sonya -s /bin/bash sonya
```

# Linux User Setup with Non-Interactive Shell
```
sudo useradd -s /sbin/nologin john
```

# Service User Creation without Home Directory
```
sudo useradd -M mark
```

# Temporary User Setup with Expiry
```
sudo useradd -e 2024-01-28 -c "Mari Yam" -d /home/mariyam -s /bin/bash mariyam
```