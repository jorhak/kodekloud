```
One of the DevOps team members was trying to install a WordPress website on a LAMP stack, which is
deployed on a Kubernetes cluster. It was working well, and we could see the installation page a few hours
ago. However, something seems to have gone wrong with the stack after the website went down. Please
look into the issue and fix it:

FYI, the deployment name is lamp-wp and it is using a service named lamp-service. Apache is using the
default HTTP port, and the NodePort is 30008. From the application logs, it has been identified that the
application is facing some issues connecting to the database, in addition to other problems. Additionally,
there are some environment variables associated with the pods, such as MYSQL_ROOT_PASSWORD,
MYSQL_DATABASE, MYSQL_USER, MYSQL_PASSWORD, and MYSQL_HOST
Also, do not attempt to delete or modify any other existing components, such as deployment names,
service names, types, labels, secrets and so on.

Note: The kubectl utility on the jump-host has been configured to work with the Kubernetes cluster.
```
# Listar objetos
```bash
kubectl get deploy,svc,pod
SUFIJO=5594699b75-lqn7q
```
Podemos ver que los objetos se estan ejecutando correctamente. Vamos a dar click sobre el boton **App**, para ver la web.
La web nos da el mensaje **502 Bad Gateway**. Procedemos a inspeccionar el **pod** y sus **contenedores**.
# Listar variables de entorno
```bash
kubectl exec -it pod/lamp-wp-$SUFIJO -c httpd-php-container -- bash
printenv | grep MYSQL_
kubectl exec -it pod/lamp-wp-$SUFIJO -c mysql-container -- bash
printenv | grep MYSQL_
```
Las variables de entorno estan bien, vamos a ver el servicio de **apache**
## Servicio apache
```bash
kubectl exec -it pod/lamp-wp-$SUFIJO -c httpd-php-container -- bash
service apache2 status
service php-fpm status
apachectl -S
```

```text
bash-4.3# service apache2 status
apache:apached                   RUNNING   pid 89, uptime 0:03:47
bash-4.3# service php-fpm status
php-fpm:php-fpmd                 RUNNING   pid 88, uptime 0:03:55
bash-4.3# apachectl -S
[Tue Nov 03 19:21:56.501219 2026] [so:warn] [pid 255:tid 137082739505992] AH01574: module socache_shmcb_module is already loaded, skipping
VirtualHost configuration:
*:80                   docker.vm (/opt/docker/etc/httpd/vhost.conf:5)
*:443                  docker.vm (/opt/docker/etc/httpd/vhost.conf:21)
ServerRoot: "/var/www"
Main DocumentRoot: "/app"
Main ErrorLog: "/docker.stderr"
Mutex ssl-stapling: using_defaults
Mutex proxy: using_defaults
Mutex ssl-cache: using_defaults
Mutex default: dir="/run/apache2/" mechanism=default 
Mutex watchdog-callback: using_defaults
Mutex ssl-stapling-refresh: using_defaults
Mutex rewrite-map: using_defaults
PidFile: "/run/apache2/httpd.pid"
Define: DUMP_VHOSTS
Define: DUMP_RUN_CFG
User: name="apache" id=100
Group: name="apache" id=101
```
El servicio se esta ejecutando correctamente. Y que **apache** se esta inicalizando desde **/opt/docker** y no desde **/etc/apache**
# Ver los logs
#### httpd
```bash
kubectl logs -f pod/lamp-wp-$SUFIJO -c httpd-php-container
```

```text
-> Executing /opt/docker/provision/entrypoint.d/05-permissions.sh
-> Executing /opt/docker/provision/entrypoint.d/20-php-fpm.sh
-> Executing /opt/docker/provision/entrypoint.d/20-php.sh
-> Executing /opt/docker/bin/service.d/supervisor.d//10-init.sh
2026-11-03 19:17:44,140 CRIT Set uid to user 0
2026-11-03 19:17:44,140 WARN Included extra file "/opt/docker/etc/supervisor.d/apache.conf" during parsing
2026-11-03 19:17:44,140 WARN Included extra file "/opt/docker/etc/supervisor.d/cron.conf" during parsing
2026-11-03 19:17:44,140 WARN Included extra file "/opt/docker/etc/supervisor.d/dnsmasq.conf" during parsing
2026-11-03 19:17:44,140 WARN Included extra file "/opt/docker/etc/supervisor.d/php-fpm.conf" during parsing
2026-11-03 19:17:44,140 WARN Included extra file "/opt/docker/etc/supervisor.d/postfix.conf" during parsing
2026-11-03 19:17:44,140 WARN Included extra file "/opt/docker/etc/supervisor.d/ssh.conf" during parsing
2026-11-03 19:17:44,140 WARN Included extra file "/opt/docker/etc/supervisor.d/syslog.conf" during parsing
2026-11-03 19:17:44,146 INFO RPC interface 'supervisor' initialized
2026-11-03 19:17:44,146 INFO supervisord started with pid 1
2026-11-03 19:17:45,147 INFO spawned: 'syslogd' with pid 87
2026-11-03 19:17:45,153 INFO spawned: 'php-fpmd' with pid 88
2026-11-03 19:17:45,155 INFO spawned: 'apached' with pid 89
2026-11-03 19:17:45,157 INFO spawned: 'crond' with pid 90
-> Executing /opt/docker/bin/service.d/syslog-ng.d//10-init.sh
2026-11-03 19:17:45,158 INFO success: php-fpmd entered RUNNING state, process has stayed up for > than 0 seconds (startsecs)
2026-11-03 19:17:45,158 INFO success: apached entered RUNNING state, process has stayed up for > than 0 seconds (startsecs)
2026-11-03 19:17:45,158 INFO success: crond entered RUNNING state, process has stayed up for > than 0 seconds (startsecs)
-> Executing /opt/docker/bin/service.d/php-fpm.d//10-init.sh
Setting php-fpm user to application
-> Executing /opt/docker/bin/service.d/httpd.d//10-init.sh
-> Executing /opt/docker/bin/service.d/cron.d//10-init.sh
[SYSLOG] syslog-ng[87]: syslog-ng starting up; version='3.7.2'
[Tue Nov 03 19:17:45.436716 2026] [so:warn] [pid 116:tid 133943296203592] AH01574: module socache_shmcb_module is already loaded, skipping
[03-Nov-2026 19:17:45] NOTICE: fpm is running, pid 88
[Tue Nov 03 19:17:45.455361 2026] [so:warn] [pid 116:tid 133943296203592] AH01574: module socache_shmcb_module is already loaded, skipping
[03-Nov-2026 19:17:45] NOTICE: ready to handle connections
[Tue Nov 03 19:17:45.457267 2026] [lbmethod_heartbeat:notice] [pid 116:tid 133943296203592] AH02282: No slotmem from mod_heartmonitor
[Tue Nov 03 19:17:45.476431 2026] [mpm_event:notice] [pid 116:tid 133943296203592] AH00489: Apache/2.4.25 (Unix) LibreSSL/2.4.4 configured -- resuming normal operations
[Tue Nov 03 19:17:45.476446 2026] [core:notice] [pid 116:tid 133943296203592] AH00094: Command line: '/usr/sbin/httpd -D FOREGROUND'
2026-11-03 19:17:46,455 INFO success: syslogd entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
```
En los logs de httpd-php-container podemos ver que el servicio se inicaliza en en **/opt/docker**. 
#### mysql
```bash
kubectl logs -f pod/lamp-wp-$SUFIJO -c mysql-container
```

```text
2026-11-03 19:17:59 1 [Note] Event Scheduler: Loaded 0 events
2026-11-03 19:17:59 1 [Note] mysqld: ready for connections.
Version: '5.6.51'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```
# Servicio
En los logs previos de httpd este se esta ejecutando desde **/opt/docker**, vamos ver los servicios de httpd y apache, porque esto puede que esten utilizando el mismo puerto, ejecutaremos un curl para ver que resultado nos devuelve.
```bash
kubectl exec -it pod/lamp-wp-$SUFIJO -c httpd-php-container -- bash
curl localhost
# no devuelve nada
curl -I localhost
```

```text
HTTP/1.1 500 Internal Server Error
Date: Tue, 03 Nov 2026 19:30:54 GMT
Server: Apache/2.4.25 (Unix)
Connection: close
Content-Type: text/html; charset=UTF-8
```
Vamos a ver los procesos que se estan ejecutando
```bash
ps aux | grep httpd
ps aux | grep apache
```

```text
bash-4.3# ps aux | grep httpd
  116 root       0:00 /usr/sbin/httpd -DFOREGROUND
  119 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  120 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  123 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  268 root       0:00 grep httpd
bash-4.3# ps aux | grep apache
   89 root       0:00 {apachectl} /bin/sh /usr/sbin/apachectl -DFOREGROUND
  119 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  120 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  123 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  270 root       0:00 grep apache
```
Podemos ver que hay dos procesos: **116** y **89**. 
El proceso **89** es el que se esta ejecutando desde **/opt/docker** y el otro proceso suponemos que es de **/etc/apache2**. Vamos a detener el servicio nataivo.
```bash
apk info apache2
service apache2 stop
ps aux | grep httpd
ps aux | grep apache
```
**apache2** esta instalado este servicio es el que vamos a detener.

```text
bash-4.3# ps aux | grep httpd
  116 root       0:00 /usr/sbin/httpd -DFOREGROUND
  119 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  120 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  123 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  281 root       0:00 grep httpd
bash-4.3# ps aux | grep apache
  119 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  120 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  123 apache     0:00 /usr/sbin/httpd -DFOREGROUND
  283 root       0:00 grep apache
```
Nos sigue devolviendo el mismo error con **curl**
```bash
curl -I localhost
```

```text
HTTP/1.1 500 Internal Server Error
Date: Tue, 03 Nov 2026 19:44:17 GMT
Server: Apache/2.4.25 (Unix)
Connection: close
Content-Type: text/html; charset=UTF-8
```

Vamos a volver a ver los logs, podemos ver que tenemos un error en **/app/index.php** en la linea 4.
```bash
kubectl logs -f pod/lamp-wp-$SUFIJO -c httpd-php-container
```

```text
-> Executing /opt/docker/bin/service.d/php-fpm.d//10-init.sh
Setting php-fpm user to application
-> Executing /opt/docker/bin/service.d/httpd.d//10-init.sh
-> Executing /opt/docker/bin/service.d/cron.d//10-init.sh
[SYSLOG] syslog-ng[87]: syslog-ng starting up; version='3.7.2'
[Tue Nov 03 19:17:45.436716 2026] [so:warn] [pid 116:tid 133943296203592] AH01574: module socache_shmcb_module is already loaded, skipping
[03-Nov-2026 19:17:45] NOTICE: fpm is running, pid 88
[Tue Nov 03 19:17:45.455361 2026] [so:warn] [pid 116:tid 133943296203592] AH01574: module socache_shmcb_module is already loaded, skipping
[03-Nov-2026 19:17:45] NOTICE: ready to handle connections
[Tue Nov 03 19:17:45.457267 2026] [lbmethod_heartbeat:notice] [pid 116:tid 133943296203592] AH02282: No slotmem from mod_heartmonitor
[Tue Nov 03 19:17:45.476431 2026] [mpm_event:notice] [pid 116:tid 133943296203592] AH00489: Apache/2.4.25 (Unix) LibreSSL/2.4.4 configured -- resuming normal operations
[Tue Nov 03 19:17:45.476446 2026] [core:notice] [pid 116:tid 133943296203592] AH00094: Command line: '/usr/sbin/httpd -D FOREGROUND'
2026-11-03 19:17:46,455 INFO success: syslogd entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
[php-fpm:access] 127.0.0.1 -  03/Nov/2026:19:30:24 +0000 "GET /index.php" 500 /app/index.php 1.124 2048 0.00%
[03-Nov-2026 19:30:24] WARNING: [pool www] child 117 said into stderr: "NOTICE: PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4"
[Tue Nov 03 19:30:24.830113 2026] [proxy_fcgi:error] [pid 119:tid 133943295195824] [client ::1:50592] AH01071: Got error 'PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4\n'
::1 - - [03/Nov/2026:19:30:24 +0000] "GET / HTTP/1.1" 500 - "-" "curl/7.52.1"
[httpd:access] localhost:80 ::1 - - [03/Nov/2026:19:30:24 +0000] "GET / HTTP/1.1" 500 bytesIn:73 bytesOut:183 reqTime:0
[php-fpm:access] 127.0.0.1 -  03/Nov/2026:19:30:54 +0000 "HEAD /index.php" 500 /app/index.php 1.033 2048 0.00%
[03-Nov-2026 19:30:54] WARNING: [pool www] child 118 said into stderr: "NOTICE: PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4"
[Tue Nov 03 19:30:54.633741 2026] [proxy_fcgi:error] [pid 123:tid 133943295195824] [client ::1:57820] AH01071: Got error 'PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4\n'
::1 - - [03/Nov/2026:19:30:54 +0000] "HEAD / HTTP/1.1" 500 - "-" "curl/7.52.1"
[httpd:access] localhost:80 ::1 - - [03/Nov/2026:19:30:54 +0000] "HEAD / HTTP/1.1" 500 bytesIn:74 bytesOut:164 reqTime:0
2026-11-03 19:41:05,981 INFO waiting for apached to stop
2026-11-03 19:41:05,981 INFO stopped: apached (terminated by SIGTERM)
[php-fpm:access] 127.0.0.1 -  03/Nov/2026:19:41:49 +0000 "GET /index.php" 500 /app/index.php 0.240 2048 0.00%
[03-Nov-2026 19:41:49] WARNING: [pool www] child 117 said into stderr: "NOTICE: PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4"
[Tue Nov 03 19:41:49.888610 2026] [proxy_fcgi:error] [pid 119:tid 133943295101616] [client ::1:51832] AH01071: Got error 'PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4\n'
::1 - - [03/Nov/2026:19:41:49 +0000] "GET / HTTP/1.1" 500 - "-" "curl/7.52.1"
[httpd:access] localhost:80 ::1 - - [03/Nov/2026:19:41:49 +0000] "GET / HTTP/1.1" 500 bytesIn:73 bytesOut:183 reqTime:0
[03-Nov-2026 19:42:17] WARNING: [pool www] child 118 said into stderr: "NOTICE: PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4"
[php-fpm:access] 127.0.0.1 -  03/Nov/2026:19:42:17 +0000 "HEAD /index.php" 500 /app/index.php 0.218 2048 0.00%
[Tue Nov 03 19:42:17.073496 2026] [proxy_fcgi:error] [pid 120:tid 133943295195824] [client ::1:45316] AH01071: Got error 'PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4\n'
::1 - - [03/Nov/2026:19:42:17 +0000] "HEAD / HTTP/1.1" 500 - "-" "curl/7.52.1"
[httpd:access] localhost:80 ::1 - - [03/Nov/2026:19:42:17 +0000] "HEAD / HTTP/1.1" 500 bytesIn:74 bytesOut:164 reqTime:0
[03-Nov-2026 19:44:17] WARNING: [pool www] child 117 said into stderr: "NOTICE: PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4"
[php-fpm:access] 127.0.0.1 -  03/Nov/2026:19:44:17 +0000 "HEAD /index.php" 500 /app/index.php 0.219 2048 0.00%
[Tue Nov 03 19:44:17.543746 2026] [proxy_fcgi:error] [pid 119:tid 133943295007408] [client ::1:44278] AH01071: Got error 'PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4\n'
::1 - - [03/Nov/2026:19:44:17 +0000] "HEAD / HTTP/1.1" 500 - "-" "curl/7.52.1"
[httpd:access] localhost:80 ::1 - - [03/Nov/2026:19:44:17 +0000] "HEAD / HTTP/1.1" 500 bytesIn:74 bytesOut:164 reqTime:0
```

#### Ingresar al container
```bash
kubectl exec -it pod/lamp-wp-$SUFIJO -c httpd-php-container -- bash
vi /app/index.php
```
#### Before
```php
<?php
$dbname = $_ENV['MYSQL_DATABASE'];
$dbuser = $_ENV['MYSQL_USER'];
$dbpass = $_ENV[''MYSQL_PASSWORD""];
$dbhost = $_ENV['MYSQL-HOST'];


$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Conneect to '$dbhost'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($test_query);

if ($result->connect_error) {
   die("Connection failed: " . $conn->connect_error);
}
  echo "Connected successfully";
```
#### After
```php
<?php
$dbname = $_ENV['MYSQL_DATABASE'];
$dbuser = $_ENV['MYSQL_USER'];
$dbpass = $_ENV['MYSQL_PASSWORD'];
$dbhost = $_ENV['MYSQL_HOST'];


$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Conneect to '$dbhost'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($test_query);

if ($result->connect_error) {
   die("Connection failed: " . $conn->connect_error);
}
  echo "Connected successfully";
```
# Verificar
```bash
curl localhost
# o
curl -I localhost
```
Presionamos el boton **App**, nos sigue saliendo **502 Bad Gateway**, vamos a inspeccionar los objetos **deploy** y **service**
```bash
kubectl get deploy/lamp-wp -o yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"lamp"},"name":"lamp-wp","namespace":"default"},"spec":{"selector":{"matchLabels":{"app":"lamp","tier":"frontend"}},"strategy":{"type":"Recreate"},"template":{"metadata":{"labels":{"app":"lamp","tier":"frontend"}},"spec":{"containers":[{"env":[{"name":"MYSQL_ROOT_PASSWORD","valueFrom":{"secretKeyRef":{"key":"password","name":"mysql-root-pass"}}},{"name":"MYSQL_DATABASE","valueFrom":{"secretKeyRef":{"key":"database","name":"mysql-db-url"}}},{"name":"MYSQL_USER","valueFrom":{"secretKeyRef":{"key":"username","name":"mysql-user-pass"}}},{"name":"MYSQL_PASSWORD","valueFrom":{"secretKeyRef":{"key":"password","name":"mysql-user-pass"}}},{"name":"MYSQL_HOST","valueFrom":{"secretKeyRef":{"key":"host","name":"mysql-host"}}}],"image":"webdevops/php-apache:alpine-3-php7","name":"httpd-php-container","ports":[{"containerPort":80,"name":"httpd"}],"volumeMounts":[{"mountPath":"/opt/docker/etc/php/php.ini","name":"php-config-volume","subPath":"php.ini"}]},{"env":[{"name":"MYSQL_ROOT_PASSWORD","valueFrom":{"secretKeyRef":{"key":"password","name":"mysql-root-pass"}}},{"name":"MYSQL_DATABASE","valueFrom":{"secretKeyRef":{"key":"database","name":"mysql-db-url"}}},{"name":"MYSQL_USER","valueFrom":{"secretKeyRef":{"key":"username","name":"mysql-user-pass"}}},{"name":"MYSQL_PASSWORD","valueFrom":{"secretKeyRef":{"key":"password","name":"mysql-user-pass"}}},{"name":"MYSQL_HOST","valueFrom":{"secretKeyRef":{"key":"host","name":"mysql-host"}}}],"image":"mysql:5.6","name":"mysql-container","ports":[{"containerPort":3306,"name":"mysql"}]}],"volumes":[{"configMap":{"name":"php-config"},"name":"php-config-volume"}]}}}}
  creationTimestamp: "2026-11-03T19:17:35Z"
  generation: 1
  labels:
    app: lamp
  name: lamp-wp
  namespace: default
  resourceVersion: "1522"
  uid: dd5183c7-5cb7-40b3-b44f-aff4d953d72f
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: lamp
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: lamp
        tier: frontend
    spec:
      containers:
      - env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql-root-pass
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database
              name: mysql-db-url
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: username
              name: mysql-user-pass
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql-user-pass
        - name: MYSQL_HOST
          valueFrom:
            secretKeyRef:
              key: host
              name: mysql-host
        image: webdevops/php-apache:alpine-3-php7
        imagePullPolicy: IfNotPresent
        name: httpd-php-container
        ports:
        - containerPort: 80
          name: httpd
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /opt/docker/etc/php/php.ini
          name: php-config-volume
          subPath: php.ini
      - env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql-root-pass
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database
              name: mysql-db-url
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: username
              name: mysql-user-pass
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql-user-pass
        - name: MYSQL_HOST
          valueFrom:
            secretKeyRef:
              key: host
              name: mysql-host
        image: mysql:5.6
        imagePullPolicy: IfNotPresent
        name: mysql-container
        ports:
        - containerPort: 3306
          name: mysql
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          defaultMode: 420
          name: php-config
        name: php-config-volume
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2026-11-03T19:17:50Z"
    lastUpdateTime: "2026-11-03T19:17:50Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2026-11-03T19:17:35Z"
    lastUpdateTime: "2026-11-03T19:17:50Z"
    message: ReplicaSet "lamp-wp-5594699b75" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```

```bash
kubectl get svc/lamp-service -o yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"lamp"},"name":"lamp-service","namespace":"default"},"spec":{"ports":[{"nodePort":30008,"port":8080}],"selector":{"app":"lamp","tier":"frontend"},"type":"NodePort"}}
  creationTimestamp: "2026-11-03T19:17:35Z"
  labels:
    app: lamp
  name: lamp-service
  namespace: default
  resourceVersion: "1486"
  uid: e2561763-87a0-431e-9e24-85129f156649
spec:
  clusterIP: 10.43.91.126
  clusterIPs:
  - 10.43.91.126
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30008
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: lamp
    tier: frontend
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```
Podemos apreciar que en el contenedor **httpd-php-container** esta utilizando el puerto **80** y el servicio **lamp-service** utiliza el puerto **8080**, vamos a editar el servicio para que los puertos sean los adecuados.
```bash
kubectl edit svc/lamp-service
```
#### Before
```
  ports:                        
  - nodePort: 30008             
    port: 8080                  
    protocol: TCP               
    targetPort: 8080
```
#### After
```
  ports:                        
  - nodePort: 30008             
    port: 80            
    protocol: TCP               
    targetPort: 80
```
Volvemos a dar click sobre **App**.