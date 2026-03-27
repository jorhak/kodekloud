## Instalar pipx
```
sudo dnf install pipx 
pipx ensurepath 
```

```
pipx install --include-deps ansible

pipx install ansible-core==4.7.0
```

Con los comandos anteriores no funciono ya que no tiene esa version en especifico, solo se tiene hasta la 2.15.13.

## Alternativa
```
sudo yum update
sudo yum install python3 python3-pip -y
```

## Instalar ansible
```
sudo pip3 install ansible==4.7.0
```

### Verificar instalacion
```
ansible --version
```

Cometi un error con al anterior instalacion con pipx 2.15.13 es equivalenta a 4.7.0 esta es la verison del comprimido y ya compilado tiene la version 2.15.13.
Lo cual es lo mismo crei que con pipx tenia versiones viejas.