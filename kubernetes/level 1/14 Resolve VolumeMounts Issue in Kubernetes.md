```
We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:



The pod name is nginx-phpfpm and configmap name is nginx-config. Identify and fix the problem.


Once resolved, copy /home/thor/index.php file from the jump host to the nginx-container within the nginx document root. After this, you should be able to access the website using Website button on the top bar.


Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```

1. Ver **Pod** y **ConfigMap**
```code
kubectl get pod/nginx-phpfpm
kubectl get cm/nginx-config
```

2. Describir **Pod** y **ConfigMap**
```code
kubectl describe pod/nginx-phpfpm
kubectl describe cm/nginx-config
```

3. Ingresar a **Pod** nginx-container:
En este contenedor tiene la ruta _/usr/share/nginx/html_
```code
kubectl exec -it pod/nginx-phpfpm -c nginx-container -- sh
```

```
curl localhost:8099
```

4. Ingresar a **Pod** php-fpm-container:
En este contenedor tiene la ruta _/var/www/html_
```code
kubectl exec -it pod/nginx-phpfpm -c php-fpm-container -- sh
```

```
curl localhost:9000
```

5. Copiar index.php 
```
kubectl cp index.php nginx-phpfpm:/usr/share/nginx/html -c nginx-container
```

6. Editar **ConfigMap**
```
kubectl edit cm/nginx-config
```

```
root /usr/share/nginx/html; ##AQUI
location ~ \.php$ {                                                
    include fastcgi_params;                                           
    fastcgi_param REQUEST_METHOD $request_method;                     
    fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script ##AQUI
    fastcgi_pass 127.0.0.1:9000;                                      
}
```

- Verificar si se hizo el cambio
```
kubectl exec -it pod/nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf
```

Como no se ve que se haga el cambio lo que debemos hacer es editar el pod para forzar el reinicio del contenedor
```
kubectl edit pod/nginx-phpfpm
```

```
####AFTER
image: nginx:latest
####BEFORE
image: nginx:la
```

```
kubectl get pod
```
Despues lo volvemos a como estaba en un principio.

Lo que vemos aqui es que tenemos un cruce de rutas por eso el cambio de **root** y **SCRIPT_FILENAME**.

