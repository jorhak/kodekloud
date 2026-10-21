```
In the `Stratos Datacenter`, our `Storage server` is encountering performance degradation due to excessive processes held by the `nfsuser` user. To mitigate this issue, we need to enforce limitations on its maximum processes. Please set the maximum process limits as specified below:  
  

  

a. Set the soft limit to `1024`  
  

b. Set the hard limit to `2024`
```

1. Ingresar al servidor
```
ssh natasha@172.16.238.15
```

2. Limitar la cantidad de procesos maximo
```
sudo vi /etc/security/limits.conf
```

```
nfsuser soft nproc 1025
nfsuser hard nproc 2024
```

Debemos salir de la sesion y volver a entrar para que se apliquen los cambios.
