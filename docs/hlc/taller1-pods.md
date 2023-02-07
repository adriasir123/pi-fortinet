# Taller 1: Trabajando con Pods

## Entregas

### Parte 1

> Pantallazo del fichero yaml que has creado con la definición del Pod

![sc1](https://i.imgur.com/GA49pxO.png)

### Parte 2

> Pantallazo donde se comprueba que el Pod ha sido creado

![sc2](https://i.imgur.com/QtIGVpZ.png)

### Parte 3

> Pantallazo donde se ve la información detallada del Pod

![sc3](https://i.imgur.com/s0t0oFb.png)

![sc3](https://i.imgur.com/TGFK0wA.png)

### Parte 4

> Pantallazo donde se ve el fichero `index.html` del DocumentRoot

![sc4](https://i.imgur.com/F89km4X.png)

### Parte 5

> Pantallazo del navegador accediendo a la aplicación con el `port-forward`

![sc5](https://i.imgur.com/DSSQmv5.png)

### Parte 6

> Pantallazo donde se vean los logs de acceso del Pod

![sc6](https://i.imgur.com/gD9kVgo.png)

## Desarrollo

### Ejercicio 1

> Crear un yaml con la descripción del recurso Pod, teniendo en cuenta los siguientes aspectos:
>
>- Indica nombres distintos para el pod y para el contenedor
>- La imagen que debes desplegar es `iesgn/test_web:latest`
>- Indica una etiqueta en la descripción del pod

```shell
nano pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-test-web
 labels:
   app: web
spec:
 containers:
   - image: iesgn/test_web:latest
     name: contenedor-test-web
     imagePullPolicy: Always
```

### Ejercicio 2

> Crear el pod

```shell
vagrant@k1:~$ kubectl create -f pod.yaml
pod/pod-test-web created
```

### Ejercicio 3

> Comprobar que el Pod se ha creado y está corriendo

```shell
vagrant@k1:~$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pod-test-web   1/1     Running   0          30m
```

### Ejercicio 4

> Obtener información detallada del Pod creado

```shell
vagrant@k1:~$ kubectl describe pod pod-test-web
Name:             pod-test-web
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.39.154
Start Time:       Tue, 07 Feb 2023 17:56:23 +0000
Labels:           app=web
Annotations:      <none>
Status:           Running
IP:               10.244.0.4
IPs:
  IP:  10.244.0.4
Containers:
  contenedor-test-web:
    Container ID:   docker://5278c71d92157f07deb321d9bd93148015cf72c4745640a80095d681c6679be3
    Image:          iesgn/test_web:latest
    Image ID:       docker-pullable://iesgn/test_web@sha256:001e1f4d8ab5d7ddf406e481392052769d1e87bdcce672fc6b91cdf3ec136886
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 07 Feb 2023 17:56:37 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xndd2 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-xndd2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <invalid>  default-scheduler  Successfully assigned default/pod-test-web to minikube
  Normal  Pulling    <invalid>  kubelet            Pulling image "iesgn/test_web:latest"
  Normal  Pulled     <invalid>  kubelet            Successfully pulled image "iesgn/test_web:latest" in 11.894282463s (11.894299468s including waiting)
  Normal  Created    <invalid>  kubelet            Created container contenedor-test-web
  Normal  Started    <invalid>  kubelet            Started container contenedor-test-web
```

### Ejercicio 5

> Acceder de forma interactiva al Pod y comprobar los ficheros que están en el DocumentRoot (`/usr/local/apache2/htdocs`)

```shell
vagrant@k1:~$ kubectl exec -it pod-test-web -- /bin/bash
root@pod-test-web:/usr/local/apache2# cd htdocs/
root@pod-test-web:/usr/local/apache2/htdocs# ls -la
total 16
drwxr-xr-x 1 root     root     4096 Feb 26  2021 .
drwxr-xr-x 1 www-data www-data 4096 Feb  9  2021 ..
-rw-r--r-- 1 root     root     2884 Feb 26  2021 index.html
```

### Ejercicio 6

> Crear una redirección con kubectl port-forward utilizando el puerto de localhost 8888 y sabiendo que el Pod ofrece el servicio en el puerto 80. Accede a la aplicación desde un navegador.

```shell
vagrant@k1:~$ kubectl port-forward pod-test-web 8888:80
Forwarding from 127.0.0.1:8888 -> 80
Forwarding from [::1]:8888 -> 80
```

![prueba](https://i.imgur.com/DSSQmv5.png)

### Ejercicio 7

> Mostrar los logs del Pod y comprobar que se visualizan los logs de los accesos que hemos realizado en el punto anterior

```shell
vagrant@k1:~$ kubectl logs pod-test-web
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.6. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.6. Set the 'ServerName' directive globally to suppress this message
[Tue Feb 07 19:32:17.378493 2023] [mpm_event:notice] [pid 1:tid 140046039004288] AH00489: Apache/2.4.46 (Unix) configured -- resuming normal operations
[Tue Feb 07 19:32:17.379568 2023] [core:notice] [pid 1:tid 140046039004288] AH00094: Command line: 'httpd -D FOREGROUND'
127.0.0.1 - - [07/Feb/2023:19:35:20 +0000] "GET / HTTP/1.1" 200 2884
127.0.0.1 - - [07/Feb/2023:19:35:21 +0000] "GET /favicon.ico HTTP/1.1" 404 196
127.0.0.1 - - [07/Feb/2023:19:36:23 +0000] "GET / HTTP/1.1" 200 2884
```

### Ejercicio 8

> Eliminar el Pod y comprobar que ha sido eliminado

```shell
vagrant@k1:~$ kubectl delete pod pod-test-web
pod "pod-test-web" deleted
vagrant@k1:~$ kubectl get pod
No resources found in default namespace.
```
