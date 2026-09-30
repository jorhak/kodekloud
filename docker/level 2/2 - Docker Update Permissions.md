```
One of the Nautilus project developers need access to run docker commands on `App Server 1`. This user is already created on the server. Accomplish this task as per details given below:  
  

  

User `james` is not able to run docker commands on `App Server 1` in Stratos DC, make the required changes so that this user can run docker commands without `sudo`.
```
# 1 Ingresamos al servidor
```
ssh tony@stapp01
```
# 2 Agregar usuario al grupo docker
```
sudo usermod -aG docker james
```
# 3 Cambiamos al usuario "james"
```
sudo su james
whoami
```
# 4 Verificar
```
docker run hello-world
```

