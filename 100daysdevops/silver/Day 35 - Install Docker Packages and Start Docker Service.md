```
The Nautilus DevOps team aims to containerize various applications following a recent meeting with the application development team. They intend to conduct testing with the following steps:


Install docker-ce and docker compose packages on App Server 1.


Initiate the docker service.
```

## Ingresar al servidor
```
ssh tony@172.16.238.10
```

## Instalar docker y docker compose
### Agregar repositorio
```
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### Instalar Docker Engine
```
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Iniciar Docker Engine
```
sudo systemctl enable --now docker
```

### Verificar estado
```
sudo systemctl status docker
```

### Version de Docker y Docker Compose
```
docker --version
docker compose version
```
