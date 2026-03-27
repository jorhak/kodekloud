```
We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:



The pod name is nginx-phpfpm and configmap name is nginx-config. Identify and fix the problem.


Once resolved, copy /home/thor/index.php file from the jump host to the nginx-container within the nginx document root. After this, you should be able to access the website using Website button on the top bar.


Note: The kubectl utility on jump_host is configured to operate with the Kubernetes cluster.
```

## Inspeccionar
```
kubectl get pod,cm
kubectl get cm nginx-config -o yaml
kubectl get pod nginx-phpfpm -o yaml
```

## Editar pod
```
kubectl edit pod nginx-phpfpm
```

```
- image: php:7.2-fpm-alpine
    imagePullPolicy: IfNotPresent
    name: php-fpm-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/www/html
      name: shared-files
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-62kxj
      readOnly: true
  - image: nginx:latest
    imagePullPolicy: Always
    name: nginx-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /usr/share/nginx/html ####CAMBIAR
      name: shared-files
    - mountPath: /etc/nginx/nginx.conf
      name: nginx-config-volume
      subPath: nginx.conf
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-62kxj
      readOnly: true
```

No nos va dejar editarlo pero nos va crear un fichero temporal en **/tmp** y vamos a editar ese fichero y vamos aplicar los cambios:
```
error: pods "nginx-phpfpm" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-29276641.yaml"
error: Edit cancelled, no valid changes were saved.
```


```
cd /tmp && ls
demofile2.json
kubectl-edit-1115096616.yaml
kubectl-edit-1916071029.yaml
kubectl-edit-1943591351.yaml
kubectl-edit-2646138055.yaml
```

```
vi kubectl-edit-<hash>.yaml
```

```
#### antes ####
volumeMounts:
    - mountPath: /usr/share/nginx/html
#### despues ####
volumeMounts:
    - mountPath: /var/www/html
```

Antes de aplicar los cambios debemos eliminar el pod actual.
```
kubectl delete pod nginx-phpfpm
```

```
kubectl apply -f kubectl-edit-<hash>.yaml
```

Con este cambio nos va dar el error de que no tenemos perimiso: **403 Forbidden** 
## Configurar permisos
```
kubectl delete pod nginx-phpfpm
vi kubectl-edit-<hash>.yaml
```

```
#### antes ####
spec:
  
#### despues ####
spec:
  securityContext:
    fsGroup: 3333
```

Volvemos a eliminar y aplicar los cambios.

## Ingresar a los contenedores (opcional)
```
kubectl exec -it nginx-phpfpm -c nginx-container -- sh
kubectl exec -it nginx-phpfpm -c php-fpm-container -- sh
```

## Copiar fichero
```
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html -c nginx-container
kubectl exec -it nginx-phpfpm -c nginx-container -- sh
cd /var/www/html
chmod 666 index.php
```
