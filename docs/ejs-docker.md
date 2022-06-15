# Ejercicios Docker

Todos estos ejercicios estarán sacados del [curso de Docker de José Domingo](https://iesgn.github.io/curso_docker_2021/).

## Introducción

### Instalación de docker

Instalo la versión de comunidad de Debian:

```console
sudo apt install docker.io
```

Hago que mi usuario sin privilegios pueda usar el cliente de docker *(para que este cambio surta efecto tendremos que cerrar sesión y volver a logearnos con nuestro usuario)*:

```console
sudo usermod -aG docker atlas
```

Pruebo que Docker se instaló:

```console
atlas@olympus:~$ docker --version
Docker version 20.10.5+dfsg1, build 55c4c88
```

### El “Hola Mundo” de docker

Creamos nuestro primer contenedor desde la imagen hello-world:

```console
atlas@olympus:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Para listar los contenedores ejecutándose:

```console
docker ps
```

Para listar *todos* los contenedores, no ejecutándose o ejecutándose:

```console
atlas@olympus:~$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
0653f26a58dc   hello-world   "/hello"   6 minutes ago   Exited (0) 6 minutes ago             blissful_ardinghelli
```

Para eliminar contenedores por id:

```console
docker rm 0653f26a58dc
```

Para eliminar contenedores por nombre:

```console
docker rm blissful_ardinghelli
```

### Ejecución simple de contenedores

Ejecutamos un echo desde una imagen Ubuntu:

```console
atlas@olympus:~$ docker run ubuntu echo 'Hello world'
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
405f018f9d1d: Pull complete
Digest: sha256:b6b83d3c331794420340093eb706a6f152d9c1fa51b262d9bf34594887c2c7ac
Status: Downloaded newer image for ubuntu:latest
Hello world
```

Comprobamos que el contenedor se ha parado:

```console
atlas@olympus:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                CREATED         STATUS                     PORTS     NAMES
45c6f4b0dbcc   ubuntu    "echo 'Hello world'"   2 minutes ago   Exited (0) 2 minutes ago             competent_davinci
```

Para listar las imágenes que ya tenemos descargadas:

```console
atlas@olympus:~$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
ubuntu        latest    27941809078c   2 days ago     77.8MB
hello-world   latest    feb5d9fea6a5   8 months ago   13.3kB
```

### Ejecutando un contenedor interactivo

```console
atlas@olympus:~$ docker run -it --name contenedor1 ubuntu bash
root@38c803b78559:/#
```

Para volver a conectarnos al contenedor después de habernos salido:

```console
atlas@olympus:~$ docker start contenedor1
contenedor1
atlas@olympus:~$ docker attach contenedor1
root@38c803b78559:/#
```

Para ejecutar comandos sobre un contenedor en ejecución:

```console
atlas@olympus:~$ docker start contenedor1
contenedor1
atlas@olympus:~$ docker exec contenedor1 ls -la
total 56
drwxr-xr-x   1 root root 4096 Jun  9 19:11 .
drwxr-xr-x   1 root root 4096 Jun  9 19:11 ..
-rwxr-xr-x   1 root root    0 Jun  9 19:11 .dockerenv
lrwxrwxrwx   1 root root    7 May 31 15:42 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Apr 18 10:28 boot
drwxr-xr-x   5 root root  360 Jun  9 19:24 dev
drwxr-xr-x   1 root root 4096 Jun  9 19:11 etc
drwxr-xr-x   2 root root 4096 Apr 18 10:28 home
lrwxrwxrwx   1 root root    7 May 31 15:42 lib -> usr/lib
lrwxrwxrwx   1 root root    9 May 31 15:42 lib32 -> usr/lib32
lrwxrwxrwx   1 root root    9 May 31 15:42 lib64 -> usr/lib64
lrwxrwxrwx   1 root root   10 May 31 15:42 libx32 -> usr/libx32
drwxr-xr-x   2 root root 4096 May 31 15:42 media
drwxr-xr-x   2 root root 4096 May 31 15:42 mnt
drwxr-xr-x   2 root root 4096 May 31 15:42 opt
dr-xr-xr-x 443 root root    0 Jun  9 19:24 proc
drwx------   1 root root 4096 Jun  9 19:13 root
drwxr-xr-x   5 root root 4096 May 31 15:45 run
lrwxrwxrwx   1 root root    8 May 31 15:42 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 May 31 15:42 srv
dr-xr-xr-x  13 root root    0 Jun  9 19:24 sys
drwxrwxrwt   2 root root 4096 May 31 15:45 tmp
drwxr-xr-x  14 root root 4096 May 31 15:42 usr
drwxr-xr-x  11 root root 4096 May 31 15:45 var
```

Para reiniciar un contenedor:

```console
atlas@olympus:~$ docker restart contenedor1
contenedor1
```

Para mostrar información sobre un contenedor:

```console
atlas@olympus:~$ docker inspect contenedor1
[
    {
        "Id": "38c803b78559b214a1e74377faf006cb1758e9bee4619c782783fd685e84b192",
        "Created": "2022-06-09T19:11:27.622457379Z",
        "Path": "bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 22633,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2022-06-09T19:27:04.276188002Z",
            "FinishedAt": "2022-06-09T19:27:03.593607062Z"
        },
        "Image": "sha256:27941809078cc9b2802deb2b0bb6feed6c236cde01e487f200e24653533701ee",
        "ResolvConfPath": "/var/lib/docker/containers/38c803b78559b214a1e74377faf006cb1758e9bee4619c782783fd685e84b192/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/38c803b78559b214a1e74377faf006cb1758e9bee4619c782783fd685e84b192/hostname",
        "HostsPath": "/var/lib/docker/containers/38c803b78559b214a1e74377faf006cb1758e9bee4619c782783fd685e84b192/hosts",
        "LogPath": "/var/lib/docker/containers/38c803b78559b214a1e74377faf006cb1758e9bee4619c782783fd685e84b192/38c803b78559b214a1e74377faf006cb1758e9bee4619c782783fd685e84b192-json.log",
        "Name": "/contenedor1",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/9fad5d5cd99a61193d036952be9856f4306065f1f2b98eec3069c1c4c25c2242-init/diff:/var/lib/docker/overlay2/4c732dfff53d5fd606c32c363b213c57a8926899ca739453ebaa448ac601efed/diff",
                "MergedDir": "/var/lib/docker/overlay2/9fad5d5cd99a61193d036952be9856f4306065f1f2b98eec3069c1c4c25c2242/merged",
                "UpperDir": "/var/lib/docker/overlay2/9fad5d5cd99a61193d036952be9856f4306065f1f2b98eec3069c1c4c25c2242/diff",
                "WorkDir": "/var/lib/docker/overlay2/9fad5d5cd99a61193d036952be9856f4306065f1f2b98eec3069c1c4c25c2242/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "38c803b78559",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "bash"
            ],
            "Image": "ubuntu",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "ddca79ac78e8d67018fe07a11ca7c316e2a74e58e11aca41ef07cb44a960aacd",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/ddca79ac78e8",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "868b39bc434bcdc04637157879fcf3b5bee0a09c5f44e6a438ddbb7f05e2802d",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "d9afc200992598b8c58b8b9387d37f9d4575203de95b19a0d9c0b643073c8759",
                    "EndpointID": "868b39bc434bcdc04637157879fcf3b5bee0a09c5f44e6a438ddbb7f05e2802d",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

Todas las imágenes tienen definidas un proceso que se ejecuta. La imagen ubuntu tiene definida por defecto el proceso bash, por lo que podríamos haber ejecutado:

```console
atlas@olympus:~$ docker run -it --name contenedor1 ubuntu
root@09f0527e808f:/#
```

### Creando un contenedor demonio

Para ejecutar un comando en un contenedor en segundo plano:

```console
atlas@olympus:~$ docker run -d --name contenedor2 ubuntu bash -c "while true; do echo hello world; sleep 1; done"
29039571153e129b1f4d901a2e9a270385fee1905c877dda3c13de53b7ad9eb4
```

Comprobar que el contenedor se sigue ejecutando:

```console
atlas@olympus:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
29039571153e   ubuntu    "bash -c 'while true…"   14 minutes ago   Up 14 minutes             contenedor2
```

Comprobar lo que está haciendo el contenedor (bucle infinito de echo):

```console
atlas@olympus:~$ docker logs contenedor2 | more
hello world
hello world
hello world
...
```

Paramos y borramos el contenedor:

```console
atlas@olympus:~$ docker stop contenedor2
contenedor2
atlas@olympus:~$ docker rm contenedor2
contenedor2
```

Un contenedor en ejecución no puede ser eliminado, hay que pararlo y borrarlo. Se puede borrar a la fuerza:

```console
docker rm -f contenedor2
```

### Creando un contenedor con un servidor web

Para lanzar un contenedor con apache en segundo plano:

```console
atlas@olympus:~$ docker run -d --name my-apache-app -p 8080:80 httpd:2.4
28bd87fd47f48201f2805dc8be0a5a4a9698dd0b4289a2ed9ae0c6f11c436775
```

Pruebo que funciona:

![dockerapache](https://i.imgur.com/66sZy1O.png)

Para ver sus logs:

```console
atlas@olympus:~$ docker logs my-apache-app
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Fri Jun 10 10:42:47.967351 2022] [mpm_event:notice] [pid 1:tid 140566825770304] AH00489: Apache/2.4.54 (Unix) configured -- resuming normal operations
[Fri Jun 10 10:42:47.967443 2022] [core:notice] [pid 1:tid 140566825770304] AH00094: Command line: 'httpd -D FOREGROUND'
172.17.0.1 - - [10/Jun/2022:10:45:34 +0000] "GET / HTTP/1.1" 200 45
```

Logs en tiempo real:

```console
docker logs -f my-apache-app
```

Para modificar el `index.html` accediendo al contenedor:

```console
atlas@olympus:~$ docker exec -it my-apache-app bash
root@eaee87a6a08f:/usr/local/apache2# cd htdocs/
root@eaee87a6a08f:/usr/local/apache2/htdocs# echo "<h1>Curso Docker</h1>" > index.html
root@eaee87a6a08f:/usr/local/apache2/htdocs# exit
exit
```

Pruebo que funciona:

![indexhtmlmodificadointeractivo](https://i.imgur.com/g5NzdPW.png)

Para modificar el `index.html` sin acceder al contenedor:

```console
docker exec my-apache-app bash -c 'echo "<h1>Curso Docker 2</h1>" > /usr/local/apache2/htdocs/index.html'
```

![indexhtmlmodificado](https://i.imgur.com/7BIsvbQ.png)

### Configuración de contenedores con variables de entorno

Para crear un contenedor con una variable:

```console
atlas@olympus:~$ docker run -it --name prueba -e USUARIO=prueba ubuntu bash
root@cf492c1f8f1a:/# echo $USUARIO
prueba
```

Para lanzar un MariaDB con una variable:

```console
atlas@olympus:~$ docker run -d --name some-mariadb -e MYSQL_ROOT_PASSWORD=my-secret-pw mariadb
Unable to find image 'mariadb:latest' locally
latest: Pulling from library/mariadb
405f018f9d1d: Already exists
7a85079b8234: Pull complete
579c7ff691b1: Pull complete
4976663b5d6d: Pull complete
169024b1fb13: Pull complete
c0ffe8ce897f: Pull complete
b583c09d23c3: Pull complete
9b9f0c08d08f: Pull complete
9cd51f984586: Pull complete
d9f506bb8aca: Pull complete
24d689f79ba4: Pull complete
Digest: sha256:88fcb7d92c7f61cd885c4d309c98461f3607aa6dbd57a2474be86e1956b36d13
Status: Downloaded newer image for mariadb:latest
fafaba08181c061d5aadbb7f79dda4e2d61cbea6dd883f80f4f51a64cecb5834
atlas@olympus:~$ docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                  NAMES
fafaba08181c   mariadb     "docker-entrypoint.s…"   14 seconds ago   Up 12 seconds   3306/tcp               some-mariadb
eaee87a6a08f   httpd:2.4   "httpd-foreground"       7 hours ago      Up 7 hours      0.0.0.0:8080->80/tcp   my-apache-app
```

Para ver la variable creada:

```console
atlas@olympus:~$ docker exec -it some-mariadb env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=fafaba08181c
TERM=xterm
MYSQL_ROOT_PASSWORD=my-secret-pw
GOSU_VERSION=1.14
MARIADB_MAJOR=10.8
MARIADB_VERSION=1:10.8.3+maria~jammy
HOME=/root
```

Para acceder a la BD:

```console
atlas@olympus:~$ docker exec -it some-mariadb bash
root@fafaba08181c:/# mysql -u root -p"$MYSQL_ROOT_PASSWORD"
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Para acceder a la BD directamente:

```console
atlas@olympus:~$ docker exec -it some-mariadb mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Para eliminar el contenedor anterior:

```console
atlas@olympus:~$ docker rm -f some-mariadb
some-mariadb
```

Para que se pueda acceder a la BD desde el exterior:

```console
atlas@olympus:~$ docker run -d -p 3306:3306 --name some-mariadb -e MYSQL_ROOT_PASSWORD=my-secret-pw mariadb
90b5f9c312cb79a57ca1397f80daa8294f75cf6fad3cf6ae3fed8e427d845fac
```

Para comprobar que ha funcionado el mapeo de puertos:

```console
atlas@olympus:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                    NAMES
90b5f9c312cb   mariadb   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:3306->3306/tcp   some-mariadb
```

Nos conectamos a la BD del contenedor desde nuestro equipo:

```console
atlas@olympus:~$ mysql -u root -p -h 127.0.0.1
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

### Ejercicios introducción

Crear un contenedor con las siguientes características:

* Demonio
* Con la imagen nginx
* Llamado servidor_web
* Acceso por puerto 8181

```console
docker run -d -p 8181:80 --name servidor_web nginx
```

Pantallazo con creación del contenedor y comprobación de que está funcionando:

![pantallazocrearnginx](https://i.imgur.com/QiKBo0w.png)

Pantallazo accediendo al servidor web:

![pantallazoaccesonginx](https://i.imgur.com/wGAqOoe.png)

Pantallazo de tus imágenes locales:

![pantallazodockerimages](https://i.imgur.com/c9wTZ4x.png)

Pantallazo eliminando el contenedor:

![pantallazoeliminandocontenedor](https://i.imgur.com/TVuRTyt.png)

Vemos en el listado final de contenedores que ya no existe "servidor_web".

## Imágenes

### Registros de imágenes: Docker Hub

Estructura de nombre de una imagen:

```console
usuario/nombre:etiqueta
```

### Gestión de imágenes

Para listar las imágenes locales:

```console
atlas@olympus:~$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
httpd         2.4       acd59370d8fb   2 days ago     144MB
mariadb       latest    ea81af801379   4 days ago     383MB
ubuntu        latest    27941809078c   4 days ago     77.8MB
nginx         latest    0e901e68141f   2 weeks ago    142MB
hello-world   latest    feb5d9fea6a5   8 months ago   13.3kB
```

Para descargar la última versión de una imagen:

```console
atlas@olympus:~$ docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
```

Para eliminar imágenes *sin contenedores creados*:

```console
atlas@olympus:~$ docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359
```

Para buscar imágenes en Docker Hub:

```console
atlas@olympus:~$ docker search hello-world
NAME                                       DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
hello-world                                Hello World! (an example of minimal Dockeriz…   1760      [OK]
kitematic/hello-world-nginx                A light-weight nginx container that demonstr…   152
tutum/hello-world                          Image to test docker deployments. Has Apache…   88                   [OK]
dockercloud/hello-world                    Hello World!                                    19                   [OK]
crccheck/hello-world                       Hello World web server in under 2.5 MB          15                   [OK]
vad1mo/hello-world-rest                    A simple REST Service that echoes back all t…   5                    [OK]
ppc64le/hello-world                        Hello World! (an example of minimal Dockeriz…   2
ansibleplaybookbundle/hello-world-db-apb   An APB which deploys a sample Hello World! a…   2                    [OK]
thomaspoignant/hello-world-rest-json       This project is a REST hello-world API to bu…   1
rancher/hello-world                                                                        1
ansibleplaybookbundle/hello-world-apb      An APB which deploys a sample Hello World! a…   1                    [OK]
businessgeeks00/hello-world-nodejs                                                         0
strimzi/hello-world-producer                                                               0
strimzi/hello-world-consumer                                                               0
koudaiii/hello-world                                                                       0
freddiedevops/hello-world-spring-boot                                                      0
strimzi/hello-world-streams                                                                0
tsepotesting123/hello-world                                                                0
armswdev/c-hello-world                     Simple hello-world C program on Alpine Linux…   0
garystafford/hello-world                   Simple hello-world Spring Boot service for t…   0                    [OK]
kevindockercompany/hello-world                                                             0
rsperling/hello-world3                                                                     0
dandando/hello-world-dotnet                                                                0
okteto/hello-world                                                                         0
uniplaces/hello-world                                                                      0
```

Para ver información sobre *imágenes locales*:

```console
atlas@olympus:~$ docker inspect hello-world
[
    {
        "Id": "sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412",
        "RepoTags": [
            "hello-world:latest"
        ],
        "RepoDigests": [
            "hello-world@sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2021-09-23T23:47:57.442225064Z",
        "Container": "8746661ca3c2f215da94e6d3f7dfdcafaff5ec0b21c9aff6af3dc379a82fbc72",
        "ContainerConfig": {
            "Hostname": "8746661ca3c2",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/hello\"]"
            ],
            "Image": "sha256:b9935d4e8431fb1a7f0989304ec86b3329a99a25f5efdc7f09f3f8c41434ca6d",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "20.10.7",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/hello"
            ],
            "Image": "sha256:b9935d4e8431fb1a7f0989304ec86b3329a99a25f5efdc7f09f3f8c41434ca6d",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 13256,
        "VirtualSize": 13256,
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/d3432ee4a8fdbdb72453b41fabf1ea7d479acb51a60649810de1561a3f241b85/merged",
                "UpperDir": "/var/lib/docker/overlay2/d3432ee4a8fdbdb72453b41fabf1ea7d479acb51a60649810de1561a3f241b85/diff",
                "WorkDir": "/var/lib/docker/overlay2/d3432ee4a8fdbdb72453b41fabf1ea7d479acb51a60649810de1561a3f241b85/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

### ¿Cómo se organizan las imágenes?

Para ver el tamaño de las imágenes:

```console
atlas@olympus:~$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
httpd         2.4       acd59370d8fb   2 days ago     144MB
mariadb       latest    ea81af801379   4 days ago     383MB
ubuntu        latest    27941809078c   4 days ago     77.8MB
nginx         latest    0e901e68141f   2 weeks ago    142MB
hello-world   latest    feb5d9fea6a5   8 months ago   13.3kB
```

Creo un contenedor interactivo y me salgo:

```console
atlas@olympus:~$ docker run -it --name contenedor1 ubuntu /bin/bash
root@d3a811e18218:/# exit
exit
```

Para visualizar los contenedores con su tamaño:

```console
atlas@olympus:~$ docker ps -a -s
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                      PORTS                    NAMES               SIZE
d3a811e18218   ubuntu    "/bin/bash"              14 minutes ago   Exited (0) 14 minutes ago                            contenedor1         5B (virtual 77.8MB)
90b5f9c312cb   mariadb   "docker-entrypoint.s…"   20 hours ago     Exited (255) 17 hours ago   0.0.0.0:3306->3306/tcp   some-mariadb        0B (virtual 383MB)
45c6f4b0dbcc   ubuntu    "echo 'Hello world'"     44 hours ago     Exited (0) 44 hours ago                              competent_davinci   0B (virtual 77.8MB)
```

El tamaño real del contenedor es casi nulo. El virtual es el que comparte con la imagen de ubuntu.

Volvemos a acceder al contenedor y creamos un fichero:

```console
atlas@olympus:~$ docker start contenedor1
contenedor1
atlas@olympus:~$ docker attach contenedor1
root@d3a811e18218:/# echo "00000000000000000">file.txt
root@d3a811e18218:/# exit
exit
```

Vemos que ha crecido el tamaño del contenedor:

```console
atlas@olympus:~$ docker ps -a -s
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                      PORTS                    NAMES               SIZE
d3a811e18218   ubuntu    "/bin/bash"              41 minutes ago   Exited (0) 6 minutes ago                             contenedor1         62B (virtual 77.8MB)
90b5f9c312cb   mariadb   "docker-entrypoint.s…"   21 hours ago     Exited (255) 17 hours ago   0.0.0.0:3306->3306/tcp   some-mariadb        0B (virtual 383MB)
45c6f4b0dbcc   ubuntu    "echo 'Hello world'"     45 hours ago     Exited (0) 44 hours ago                              competent_davinci   0B (virtual 77.8MB)
```

Para ver información sobre las capas de una imagen:

```console
docker inspect ubuntu:latest
```

![layersubuntu](https://i.imgur.com/HbZu9jM.png)

### Creación de contenedores desde imágenes

Todas las imágenes tienen definidas un proceso que se ejecuta por defecto, pero en la mayoría de los casos podemos indicar un proceso al crear un contenedor.

En la imagen ubuntu el proceso por defecto es bash, por lo tanto podemos ejecutar:

```console
atlas@olympus:~$ docker run -it --name contenedor1 ubuntu
root@190f78badf0e:/#
```

También podemos indicar el comando a ejecutar al crear un contenedor:

```console
atlas@olympus:~$ docker run ubuntu /bin/echo 'Hello world'
Hello world
```

La imagen httpd:2.4 ejecuta un servidor web por defecto:

```console
atlas@olympus:~$ docker run -d --name my-apache-app -p 8080:80 httpd:2.4
d5d49317f2a978a8ee163db794f43a6bc09b1594444fd9a575e80ddecf04e469
```

### Ejemplo: Desplegando la aplicación MediaWiki

Para instalar la última versión:

```console
docker run -d -p 8080:80 --name mediawiki1 mediawiki
```

Comprobamos que funciona y es efectivamente la última versión:

![mediawikilatest](https://i.imgur.com/fLQW5ct.png)

Instalamos otra versión de MediaWiki, la 1.36.3:

```console
docker run -d -p 8081:80 --name mediawiki2 mediawiki:1.36.3
```

Comprobamos que funciona y es efectivamente la versión indicada:

![mediawiki1.36.3](https://i.imgur.com/XZKlxAo.png)

Finalmente instalamos otra versión:

```console
docker run -d -p 8082:80 --name mediawiki3 mediawiki:1.35.5
```

Comprobamos que funciona y es efectivamente la versión indicada:

![mediawiki1.35.5](https://i.imgur.com/nJ6wArl.png)

A lo largo de este ejemplo hemos podido comprobar que las capas que ya existen no se las vuelve a descargar.

### Ejercicios imágenes

#### Servidor web

Lanzar un contenedor con características:

* Imagen php:7.4-apache
* Llamado web
* Puerto 8000

```console
atlas@olympus:~$ docker run -d -p 8000:80 --name web php:7.4-apache
Unable to find image 'php:7.4-apache' locally
7.4-apache: Pulling from library/php
42c077c10790: Already exists
8934009a9160: Already exists
5357ac116991: Already exists
54ae63894b5a: Already exists
772088206f85: Already exists
3b81c5474649: Already exists
c62a528527ae: Already exists
344c792709d4: Already exists
ef9638273c35: Already exists
4ec70638e42f: Already exists
5a0f65f5bb3b: Already exists
55578a5ce5e8: Already exists
f1dd85b4dfa5: Already exists
Digest: sha256:c65cd02abef32e0ab1558788dbcf855e6dde301159da14651c844f533b34a411
Status: Downloaded newer image for php:7.4-apache
a46be0b60d37a4e009c0b40934aafa72332a1da8e04c6dec8712b8354630d161
```

Muestro que se está ejecutando:

```console
atlas@olympus:~$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                  NAMES
a46be0b60d37   php:7.4-apache   "docker-php-entrypoi…"   3 minutes ago   Up 3 minutes   0.0.0.0:8000->80/tcp   web
```

Colocar en `/var/www/html` un `index.html` con contenido `<h1>HOLA SOY ADRIAN JARAMILLO</h1>`:

```console
docker exec web bash -c 'echo "<h1>HOLA SOY ADRIAN JARAMILLO</h1>" > /var/www/html/index.html'
```

Muestro que ha funcionado:

![indexhtmlmodificado](https://i.imgur.com/t5ACWLl.png)

Colocar en `/var/www/html` un `index.php` con contenido `<?php echo phpinfo(); ?>`:

```console
docker exec web bash -c 'echo "<?php echo phpinfo(); ?>" > /var/www/html/index.php'
```

Muestro que ha funcionado:

![indexphpañadido](https://i.imgur.com/FnvTKhl.png)

Muestro el tamaño del contenedor `web` tras crear los dos ficheros:

![tamañoweb+ficheros](https://i.imgur.com/e8rCVhr.png)

#### Servidor BD

Lanzar un contenedor con características:

* Llamado bbdd
* Imagen mariadb
* Puerto 3336
* Contraseña root: root
* Base de datos: prueba
* Usuario: invitado
* Contraseña: invitado

```console
docker run -d -p 3336:3306 --name bbdd -e MARIADB_ROOT_PASSWORD=root -e MARIADB_DATABASE=prueba -e MARIADB_USER=invitado -e MARIADB_PASSWORD=invitado mariadb
```

Mostrar el acceso a la BD desde nuestro ordenador con el usuario invitado, y mostrar la BD prueba:

![accesobdinvitado](https://i.imgur.com/sshonuY.png)

Mostrar que no se puede borrar la imagen mariadb mientras el contenedor `bbdd` existe:

![intentoborradomariadbimage](https://i.imgur.com/gM8Dumo.png)

## Almacenamiento

### Los contenedores son efímeros

Los datos de los contenedores sobreviven a paradas pero no a destrucciones.

Ejemplo:

```console
atlas@olympus:~$ docker run -d --name my-apache-app -p 8080:80 httpd:2.4
4dbe1355067367db8a435256af733a1405cab06de21b61cd67c269d0891eb48c
atlas@olympus:~$ docker exec my-apache-app bash -c 'echo "<h1>Hola</h1>" > /usr/local/apache2/htdocs/index.html'
atlas@olympus:~$ curl http://localhost:8080
<h1>Hola</h1>
atlas@olympus:~$ docker rm -f my-apache-app
my-apache-app
atlas@olympus:~$ docker run -d --name my-apache-app -p 8080:80 httpd:2.4
be4204135657e6ddbeccc2d8205e5977b0fc771bddf932a21a3e0bc9b5df2b68
atlas@olympus:~$ curl http://localhost:8080
<html><body><h1>It works!</h1></body></html>
```

Nos damos cuenta de que el `index.html` modificado se pierde, y al lanzar otro contenedor vuelve al contenido original.

### Volúmenes docker y bind mount

#### Volúmenes

Se guardan en `/var/lib/docker/volumes`

Para crear un volumen:

```console
atlas@olympus:~$ docker volume create my-vol
my-vol
```

Para eliminar un volumen:

```console
atlas@olympus:~$ docker volume rm my-vol
my-vol
```

Para eliminar volúmenes que no están siendo usados por ningún contenedor:

```console
atlas@olympus:~$ docker volume prune
WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Volumes:
ee50d8c9aa7de5fda336d094a3bb25566667f88a4c9864e3b1fd8e3bd619b0d8
6a3bd1b8cbc0029c01daf50080d3d682918e6f3fed56164914a81df7c09535dd

Total reclaimed space: 295MB
```

Para listar volúmenes:

```console
atlas@olympus:~$ docker volume ls
DRIVER    VOLUME NAME
local     f108fc0c5812145adaa6d4987a0afd9748876ff8bd7b5520e916eb530a3d35c3
```

Para ver información sobre volúmenes:

```console
atlas@olympus:~$ docker volume inspect f108fc0c5812145adaa6d4987a0afd9748876ff8bd7b5520e916eb530a3d35c3
[
    {
        "CreatedAt": "2022-06-12T13:34:02+02:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/f108fc0c5812145adaa6d4987a0afd9748876ff8bd7b5520e916eb530a3d35c3/_data",
        "Name": "f108fc0c5812145adaa6d4987a0afd9748876ff8bd7b5520e916eb530a3d35c3",
        "Options": null,
        "Scope": "local"
    }
]
```

#### Bind mounts

Esto es mapear/montar directorios/ficheros de mi sistema de ficheros con el contenedor.

### Asociando almacenamiento a los contenedores: volúmenes docker

Ya sean volúmenes o bind mount, usaremos las mismas flags de `docker run`:

* `--volume` o `-v`
* `--mount`

Creamos un volumen:

```console
atlas@olympus:~$ docker volume create miweb
miweb
```

Lanzamos un contenedor con el volumen asociado y modificamos el `index.html`:

```console
atlas@olympus:~$ docker run -d --name my-apache-app --mount type=volume,src=miweb,dst=/usr/local/apache2/htdocs -p 8080:80 httpd:2.4
4192e7c4df28ab3a7df706fa5eaad3c9fd893379a8be9705e1b714de80d284c2
atlas@olympus:~$ docker exec my-apache-app bash -c 'echo "<h1>Hola</h1>" > /usr/local/apache2/htdocs/index.html'
atlas@olympus:~$ curl http://localhost:8080
<h1>Hola</h1>
atlas@olympus:~$ docker rm -f my-apache-app
my-apache-app
```

Tras borrarlo creamos otro con el volumen asociado, pero usamos `-v`:

```console
atlas@olympus:~$ docker run -d --name my-apache-app -v miweb:/usr/local/apache2/htdocs -p 8080:80 httpd:2.4
a115083f365e64e138c2fd9adf944382abff30318f685b75c8368d1d2802a89b
```

Comprobamos que no se ha perdido la información:

```console
atlas@olympus:~$ curl http://localhost:8080
<h1>Hola</h1>
```

Si no indicamos volumen se crea uno nuevo:

```console
atlas@olympus:~$ docker volume ls
DRIVER    VOLUME NAME
local     f108fc0c5812145adaa6d4987a0afd9748876ff8bd7b5520e916eb530a3d35c3
atlas@olympus:~$ docker run -d --name my-apache-app --mount type=volume,dst=/usr/local/apache2/htdocs -p 8080:80 httpd:2.4
151f85c244bb81b8b26734c612b3b584f76dd3638b00210a0d6b30ecf4b35066
atlas@olympus:~$ docker volume ls
DRIVER    VOLUME NAME
local     65eb5fc7fc9f541f1dd35effc2e456e615f6ef51670ef07a9971d2278c9dd4ee
local     f108fc0c5812145adaa6d4987a0afd9748876ff8bd7b5520e916eb530a3d35c3
```

Si usamos `-v` y decimos un nombre se crea un volumen nuevo:

```console
atlas@olympus:~$ docker run -d --name my-apache-app -v wwwroot:/usr/local/apache2/htdocs -p 8080:80 httpd:2.4
c22f12f358f1f066c33d067b3492200829cce6af11cbde07724b8058caebfc3b
atlas@olympus:~$ docker volume ls
DRIVER    VOLUME NAME
local     65eb5fc7fc9f541f1dd35effc2e456e615f6ef51670ef07a9971d2278c9dd4ee
local     f108fc0c5812145adaa6d4987a0afd9748876ff8bd7b5520e916eb530a3d35c3
local     wwwroot
```

Tanto volúmenes como bind mount sobreescribirán la carpeta destino del contenedor.

### Asociando almacenamiento a los contenedores: bind mount

Creamos un directorio en nuestra máquina con un `index.html`:

```console
atlas@olympus:~$ mkdir web
atlas@olympus:~$ cd web
atlas@olympus:~/web$ echo "<h1>Hola</h1>" > index.html
```

Montamos ese directorio en un contenedor con `-v`:

```console
atlas@olympus:~/web$ docker run -d --name my-apache-app -v /home/atlas/web:/usr/local/apache2/htdocs -p 8080:80 httpd:2.4
426d7ef55bb658c87964fd7a26589e6325163549f1053930076b0696e0751550
```

Comprobamos que estamos sirviendo el fichero de nuestro directorio:

```console
atlas@olympus:~/web$ curl http://localhost:8080
<h1>Hola</h1>
```

Eliminamos el contenedor y creamos otro con el directorio montado, esta vez usando `--mount`:

```console
atlas@olympus:~$ docker rm -f my-apache-app
my-apache-app
atlas@olympus:~$ docker run -d --name my-apache-app --mount type=bind,src=/home/atlas/web,dst=/usr/local/apache2/htdocs -p 8080:80 httpd:2.4
d892a24a44f42d967a3cffb218641d29812fbf258c158a676cf3fff821f9eb7e
atlas@olympus:~$ curl http://localhost:8080
<h1>Hola</h1>
```

Podemos modificar el `index.html` aunque esté montado en el contenedor:

```console
atlas@olympus:~$ echo "<h1>Adios</h1>" > web/index.html
atlas@olympus:~$ curl http://localhost:8080
<h1>Adios</h1>
```

Si usamos bind mount con `-v` y la carpeta origen no existe, se crea, pero estará vacía.  
Si usamos bind mount con `--mount` y la carpeta origen no existe, dará un error.

### Ejemplo 1: Contenedor Nextcloud con almacenamiento persistente

#### Con volumen

Creo uno:

```console
atlas@olympus:~$ docker volume create nextcloud
nextcloud
```

Lanzo un contenedor enlazado con el volumen:

```console
atlas@olympus:~$ docker run -d -p 80:80 -v nextcloud:/var/www/html --name contenedor_nextcloud nextcloud
cc2e756082a06500578913dbaffdf36d6cd92e18d029f2a737737c1455a80b7d
```

Accedemos, hacemos la instalación, y subimos un fichero:

![ficheronextcloud](https://i.imgur.com/5NYKH9A.png)

Eliminamos el contenedor y creamos otro con el mismo volumen:

```console
atlas@olympus:~$ docker rm -f contenedor_nextcloud
contenedor_nextcloud
atlas@olympus:~$ docker run -d -p 80:80 -v nextcloud:/var/www/html --name contenedor_nextcloud nextcloud
b59bf01aa8794df605ad0546571bfbc1204cb4e2dc1b74201954c122e84f5206
```

Accedemos de nuevo y comprobamos que todo sigue igual:

![nextcloudrelanzado](https://i.imgur.com/avfGs1F.png)

#### Con bind mount

Crearemos un directorio que luego montaremos en el contenedor:

```console
mkdir datos_nextcloud
```

Creamos el contenedor:

```console
atlas@olympus:~$ docker run -d -p 80:80 -v /home/atlas/datos_nextcloud:/var/www/html --name contenedor_nextcloud nextcloud
ee29347f1eafd00ba329333b232bb9020457783de5fe9d1ae561fa7fb63c85f8
```

Volvemos a acceder, instalamos, y subimos un fichero:

![ficheroenbindmount](https://i.imgur.com/aGMn8P4.png)

Al usar bind mount tenemos acceso al directorio:

```console
atlas@olympus:~$ cd datos_nextcloud/
atlas@olympus:~/datos_nextcloud$ ls -la
total 184
drwxr-xr-x 15 www-data root   4096 Jun 12 19:34 .
drwxr-xr-x 38 atlas    atlas  4096 Jun 12 19:30 ..
drwxr-xr-x 43 www-data root   4096 Jun 12 19:34 3rdparty
drwxr-xr-x 48 www-data root   4096 Jun 12 19:34 apps
-rw-r--r--  1 www-data root  19327 Jun 12 19:34 AUTHORS
drwxr-xr-x  2 www-data root   4096 Jun 12 19:38 config
-rw-r--r--  1 www-data root   3924 Jun 12 19:34 console.php
-rw-r--r--  1 www-data root  34520 Jun 12 19:34 COPYING
drwxr-xr-x 22 www-data root   4096 Jun 12 19:34 core
-rw-r--r--  1 www-data root   6260 Jun 12 19:34 cron.php
drwxr-xr-x  2 www-data root   4096 Jun 12 19:34 custom_apps
drwxrwx---  5 www-data root   4096 Jun 12 19:41 data
drwxr-xr-x  2 www-data root  12288 Jun 12 19:34 dist
-rw-r--r--  1 www-data root   4387 Jun 12 19:38 .htaccess
-rw-r--r--  1 www-data root    156 Jun 12 19:34 index.html
-rw-r--r--  1 www-data root   3456 Jun 12 19:34 index.php
drwxr-xr-x  6 www-data root   4096 Jun 12 19:34 lib
-rwxr-xr-x  1 www-data root    283 Jun 12 19:34 occ
drwxr-xr-x  2 www-data root   4096 Jun 12 19:34 ocm-provider
drwxr-xr-x  2 www-data root   4096 Jun 12 19:34 ocs
drwxr-xr-x  2 www-data root   4096 Jun 12 19:34 ocs-provider
-rw-r--r--  1 www-data root   3139 Jun 12 19:34 public.php
-rw-r--r--  1 www-data root   5340 Jun 12 19:34 remote.php
drwxr-xr-x  4 www-data root   4096 Jun 12 19:34 resources
-rw-r--r--  1 www-data root     26 Jun 12 19:34 robots.txt
-rw-r--r--  1 www-data root   2452 Jun 12 19:34 status.php
drwxr-xr-x  3 www-data root   4096 Jun 12 19:34 themes
-rw-r--r--  1 www-data root    101 Jun 12 19:34 .user.ini
-rw-r--r--  1 www-data root    382 Jun 12 19:34 version.php
```

Si eliminamos el contenedor y lo volvemos a crear con el mismo directorio bind mount, todo seguirá igual:

```console
atlas@olympus:~$ docker rm -f contenedor_nextcloud
contenedor_nextcloud
atlas@olympus:~$ docker run -d -p 80:80 -v /home/atlas/datos_nextcloud:/var/www/html --name contenedor_nextcloud nextcloud
74099565d46f4ddb28547a071349a487bd511ac5e2cc47b3f29d72a6cfedd7ab
```

![nextcloudbindmountrecreado](https://i.imgur.com/yAgZWlb.png)

### Ejemplo 2: Contenedor MariaDB con almacenamiento persistente

Lanzamos el contenedor:

```console
atlas@olympus:~$ docker run --name some-mariadb -v /home/atlas/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mariadb
d12a51b521342a15b3b82cd59bab2d9de15946e5cd77a1d9149e892610121224
```

El directorio `/home/atlas/datadir` se creará automáticamente.  
Si borramos el contenedor, lo volvemos a crear, e indicamos ese directorio como bind mount, todo seguirá igual.

```console
atlas@olympus:~$ cd datadir/
atlas@olympus:~/datadir$ ls -la
total 139260
drwxr-xr-x  5 systemd-coredump systemd-coredump      4096 Jun 12 20:04 .
drwxr-xr-x 39 atlas            atlas                 4096 Jun 12 20:04 ..
-rw-rw----  1 systemd-coredump systemd-coredump  16719872 Jun 12 20:04 aria_log.00000001
-rw-rw----  1 systemd-coredump systemd-coredump        52 Jun 12 20:04 aria_log_control
-rw-rw----  1 systemd-coredump systemd-coredump         9 Jun 12 20:04 ddl_recovery.log
-rw-rw----  1 systemd-coredump systemd-coredump       868 Jun 12 20:04 ib_buffer_pool
-rw-rw----  1 systemd-coredump systemd-coredump  12582912 Jun 12 20:04 ibdata1
-rw-rw----  1 systemd-coredump systemd-coredump 100663296 Jun 12 20:06 ib_logfile0
-rw-rw----  1 systemd-coredump systemd-coredump  12582912 Jun 12 20:04 ibtmp1
-rw-rw----  1 systemd-coredump systemd-coredump         0 Jun 12 20:04 multi-master.info
drwx------  2 systemd-coredump systemd-coredump      4096 Jun 12 20:04 mysql
-rw-r--r--  1 systemd-coredump systemd-coredump        14 Jun 12 20:04 mysql_upgrade_info
drwx------  2 systemd-coredump systemd-coredump      4096 Jun 12 20:04 performance_schema
drwx------  2 systemd-coredump systemd-coredump     12288 Jun 12 20:04 sys
atlas@olympus:~/datadir$ docker exec -it some-mariadb bash -c 'mysql -uroot -p$MYSQL_ROOT_PASSWORD'
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database prueba;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> quit
Bye
atlas@olympus:~/datadir$ docker rm -f some-mariadb
some-mariadb
atlas@olympus:~/datadir$ docker run --name some-mariadb -v /home/atlas/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mariadb
84fb33568b2db708442f4099d6a8cf2636a2edb676630acbb7e71ca33f4ae2cc
atlas@olympus:~/datadir$ docker exec -it some-mariadb bash -c 'mysql -uroot -p$MYSQL_ROOT_PASSWORD'
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| prueba             |
| sys                |
+--------------------+
5 rows in set (0.001 sec)

MariaDB [(none)]>
```

### Otros usos del almacenamiento

#### Compartir información entre contenedores

Tendremos un contenedor principal con un servidor web sirviendo un `index.html`. El segundo contenedor  irá actualizándolo.

Creamos el volumen:

```console
atlas@olympus:~$ docker volume create datos_compartidos
datos_compartidos
```

Lanzamos el primer contenedor con el volumen montado en su `DocumentRoot`, en solo lectura *(ro)*:

```console
atlas@olympus:~$ docker run -d -p 8181:80 --name contenedor1 -v datos_compartidos:/var/www/html:ro php:7.4-apache
2c2eb3c74624156a94360ec93b431193a4eb39d9f8ce51f54ad5c7618e9b4fe5
```

Lanzamos el segundo contenedor, que irá modificando `index.html` en el volumen cada segundo:

```console
atlas@olympus:~$ docker run -d --name contenedor2 -v datos_compartidos:/srv debian bash -c "while true; do date >> /srv/index.html;sleep 1;done"
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
e756f3fdd6a3: Pull complete
Digest: sha256:3f1d6c17773a45c97bd8f158d665c9709d7b29ed7917ac934086ad96f92e4510
Status: Downloaded newer image for debian:latest
33aac5ea223daf385d736bc43ea73b5e3dc0f033c050b968224b9f2a5d187f99
```

Accedemos al servidor web y comprobamos que funciona:

![indexhtmlautomatico](https://i.imgur.com/ZeBJwbn.png)

También se podría haber hecho con bind mount:

```console
atlas@olympus:~$ docker run -d -p 8181:80 --name contenedor1 -v /home/atlas/compartido:/var/www/html:ro php:7.4-apache
af6c26e17a3857d54c3c35363622412cb2381bed3f1c981add1ff4ea10713f97
atlas@olympus:~$ docker run -d --name contenedor2 -v /home/atlas/compartido:/srv debian bash -c "while true; do date >> /srv/index.html;sleep 1;done"
f4b562b1e9c38afb49b0e830e9dc9f78d3d964b7741b6937282672f7a1256247
```

Comprobamos que sigue funcionando:

![indexhtmlautomaticobindmount](https://i.imgur.com/EZOOOyZ.png)

Podríamos ver el fichero `~/compartido/index.html` en nuestra máquina:

```console
atlas@olympus:~$ ls -la compartido/
total 20
drwxr-xr-x  2 root  root  4096 Jun 12 21:04 .
drwxr-xr-x 40 atlas atlas 4096 Jun 12 21:03 ..
-rw-r--r--  1 root  root  9280 Jun 12 21:09 index.html
```

#### Comprobar compatibilidad de código entre distintas versiones de un lenguaje de programación

Creamos el directorio `codigo`:

```console
mkdir codigo
```

Dentro creamos un `index.php` con el siguiente código:

```console
<?php
// Funciona bien en php5 ya que list hace la asignaciÃ³n desde el Ãºltimo al primero
$info = array('cafeÃ­na','marrÃ³n', 'cafÃ©');

// Enumerar todas las variables
list($datos[], $datos[], $datos[]) = $info;
echo "El $datos[0] es $datos[1] y la $datos[2] lo hace especial.\n";
?>
```

Lanzamos dos contenedores sirviendo este código pero con imágenes distintas y puertos distintos:

```console
atlas@olympus:~$ docker run -d -p 8081:80 --name php56 -v /home/atlas/codigo:/var/www/html:ro php:5.6-apache
Unable to find image 'php:5.6-apache' locally
5.6-apache: Pulling from library/php
5e6ec7f28fb7: Pull complete
cf165947b5b7: Pull complete
7bd37682846d: Pull complete
99daf8e838e1: Pull complete
ae320713efba: Pull complete
ebcb99c48d8c: Pull complete
9867e71b4ab6: Pull complete
936eb418164a: Pull complete
bc298e7adaf7: Pull complete
ccd61b587bcd: Pull complete
b2d4b347f67c: Pull complete
56e9dde34152: Pull complete
9ad99b17eb78: Pull complete
Digest: sha256:0a40fd273961b99d8afe69a61a68c73c04bc0caa9de384d3b2dd9e7986eec86d
Status: Downloaded newer image for php:5.6-apache
368c1d95a662014540b74505b892d5f0e9df214aa6708c9a10c132bb6ce03972
atlas@olympus:~$ docker run -d -p 8082:80 --name php74 -v /home/atlas/codigo:/var/www/html:ro php:7.4-apache
ae94d1294cdc62c9af7d71c383c63be295ede4eb626907cd037124a8b5f8fc47
```

Ya podemos acceder a cada puerto y comparar:

![codigophp5](https://i.imgur.com/V6DAndH.png)

![codigophp7](https://i.imgur.com/c3hHOTM.png)

### Ejercicios almacenamiento

Creo los siguientes volúmenes:

```console
atlas@olympus:~$ docker volume create volumen_datos
volumen_datos
atlas@olympus:~$ docker volume create volumen_web
volumen_web
```

![volumenescreados](https://i.imgur.com/m0Gw3Uv.png)

Lanzo el primer contenedor:

```console
docker run -d -p 8080:80 --name c1 -v volumen_web:/var/www/html php:7.4-apache
```

![c1lanzado](https://i.imgur.com/HNL1LE1.png)

Lanzo el segundo contenedor:

```console
docker run -d --name c2 -v volumen_datos:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=admin mariadb
```

![c2lanzado](https://i.imgur.com/5MVBdjr.png)

Paramos y borramos `c2`, luego borramos `volumen_datos`:

```console
atlas@olympus:~$ docker stop c2
c2
atlas@olympus:~$ docker rm c2
c2
atlas@olympus:~$ docker volume rm volumen_datos
volumen_datos
```

![volumendatosborrado](https://i.imgur.com/s9tvCBv.png)

Creamos un `index.html` en `c1` y comprobamos que funciona:

```console
docker exec c1 bash -c 'echo "<h1>Web C1</h1>" > /var/www/html/index.html'
```

![indexhtmlc1](https://i.imgur.com/u1IpP0A.png)

Borramos `c1` y creamos `c3`:

```console
atlas@olympus:~$ docker stop c1
c1
atlas@olympus:~$ docker rm c1
c1
atlas@olympus:~$ docker run -d -p 8081:80 --name c3 -v volumen_web:/var/www/html php:7.4-apache
7c401c786a6aac6aa4c1ec8874066fb63b943554b273637e996ef4d5b6fc6e2a
```

![creadoc3](https://i.imgur.com/Hetp5OS.png)

Comprobamos que el `index.html` anterior sigue funcionando en `c3`:

![c3funcionando](https://i.imgur.com/t0gpHKn.png)

## Redes

### Introducción a las redes en Docker

Lanzo un contenedor que se borrará al pararse *(--rm)*:

```console
atlas@olympus:~$ docker run -it --name contenedor1 --rm debian bash
root@222b4c0ff2cd:/#
```

Para ver la IP asignada:

```console
atlas@olympus:~$ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' contenedor1
172.17.0.2
```

El bridge `docker0` en nuestro host es al que se conectan todos los contenedores por defecto:

![docker0mostrado](https://i.imgur.com/sihaLm1.png)

Docker nos modifica el cortafuegos:

```console
atlas@olympus:~$ sudo iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy DROP)
target     prot opt source               destination
DOCKER-USER  all  --  0.0.0.0/0            0.0.0.0/0
DOCKER-ISOLATION-STAGE-1  all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain DOCKER (1 references)
target     prot opt source               destination

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target     prot opt source               destination
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0
RETURN     all  --  0.0.0.0/0            0.0.0.0/0

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
target     prot opt source               destination
DROP       all  --  0.0.0.0/0            0.0.0.0/0
RETURN     all  --  0.0.0.0/0            0.0.0.0/0

Chain DOCKER-USER (1 references)
target     prot opt source               destination
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
```

```console
atlas@olympus:~$ sudo iptables -L -n -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
```

### Tipos de redes en Docker

Para listar las redes:

```console
atlas@olympus:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cefa7428778f   bridge    bridge    local
34d0a5b2368f   host      host      local
9e2bf2980f1b   none      null      local
```

Por defecto los contenedores se conectan a `bridge`:

![dockerbridgenet](https://iesgn.github.io/curso_docker_2021/sesion4/img/bridge1.png)

En la tipo `host` el contenedor ofrece los servicios en la IP y puertos del anfitrión. No tiene ip propia:

```console
atlas@olympus:~$ docker run -d --name mi_servidor --network host josedom24/aplicacionweb:v1
Unable to find image 'josedom24/aplicacionweb:v1' locally
v1: Pulling from josedom24/aplicacionweb
c5e155d5a1d1: Pull complete
f2b76f8f3462: Pull complete
2e8f82335e4d: Pull complete
Digest: sha256:6c28c9bbb10ea66f08c27377bf35cbf23b6bf64f1cc71b20dcd0d39686835796
Status: Downloaded newer image for josedom24/aplicacionweb:v1
89950cbe5d6f29c9f24fac00b720cf70671396f74770d2c2d36f14ae626b7a31
atlas@olympus:~$ docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS     NAMES
89950cbe5d6f   josedom24/aplicacionweb:v1   "/usr/sbin/apache2ct…"   37 seconds ago   Up 36 seconds             mi_servidor
```

Podemos acceder localmente al contenedor, sin mapeo:

![webentiponetworkhost](https://i.imgur.com/XaixYm8.png)

La red `none` no asigna IP y aísla a los contenedores, solo da loopback. Se usa para batch jobs.

### Gestionando las redes en docker

Bridge `docker0` por defecto ≠ Bridges definidos por el usuario.

Los contenedores en producción **deben estar en bridges de usuario, son mejores**.

Para listar redes:

```console
atlas@olympus:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cefa7428778f   bridge    bridge    local
34d0a5b2368f   host      host      local
9e2bf2980f1b   none      null      local
```

Para crear redes:

```console
atlas@olympus:~$ docker network create red1
dcec4de396e9802627a000d813e5cc65da4f0992a9cf5ace62eff12f13735da5
atlas@olympus:~$ docker network create -d bridge --subnet 172.24.0.0/16 --gateway 172.24.0.1 red2
e6cacc4df914f5cb9de6b336679ec1ed7e804407f0e56968c74a4a3e7e7a8deb
atlas@olympus:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cefa7428778f   bridge    bridge    local
34d0a5b2368f   host      host      local
9e2bf2980f1b   none      null      local
dcec4de396e9   red1      bridge    local
e6cacc4df914   red2      bridge    local
```

Para borrar redes:

```console
atlas@olympus:~$ docker network rm red1
red1
atlas@olympus:~$ docker network prune
WARNING! This will remove all custom networks not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Networks:
red2
```

Para ver información de una red:

```console
atlas@olympus:~$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "cefa7428778f2b0a5af1617a3659829650226ea874cb037d71efd441d80a8a79",
        "Created": "2022-06-13T17:40:22.758388059+02:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

Cada red que creamos genera un bridge como podemos ver:

![bridgegenerado](https://i.imgur.com/svKbyC9.png)

### Uso de la red bridge por defecto

Manera no recomendada. Usaremos el obsoleto `--link`.

Lanzo un contenedor:

```console
atlas@olympus:~$ docker run -d --name servidor_mariadb \
                -e MYSQL_DATABASE=mi_basededatos \
                -e MYSQL_USER=usuario \
                -e MYSQL_PASSWORD=asdasd \
                -e MYSQL_ROOT_PASSWORD=asdasd \
                mariadb
026b003f91c9289fd2d0c1da284057b77607ab62b218321630e887675fddf148
```

Lanzo otro contenedor enlazado con el anterior:

```console
atlas@olympus:~$ docker run -d --name servidor_web --link servidor_mariadb:mariadb nginx
6336b40e133735e408e5b043763e5eeefea2c51f35a3447d3167fcfed17d6a0f
```

El enlace se hace por resolución estática:

```console
atlas@olympus:~$ docker exec servidor_web cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0ip6-localnet
ff00::0ip6-mcastprefix
ff02::1ip6-allnodes
ff02::2ip6-allrouters
172.17.0.2	mariadb 026b003f91c9 servidor_mariadb
172.17.0.3	6336b40e1337
```

El DNS del contenedor es el mismo que el de nuestro host = las resoluciones no se hacen con un DNS:

```console
atlas@olympus:~$ docker exec servidor_web cat /etc/resolv.conf
# Generated by NetworkManager
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Se comparten variables:

```console
atlas@olympus:~$ docker exec servidor_web env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=6336b40e1337
MARIADB_PORT=tcp://172.17.0.2:3306
MARIADB_PORT_3306_TCP=tcp://172.17.0.2:3306
MARIADB_PORT_3306_TCP_ADDR=172.17.0.2
MARIADB_PORT_3306_TCP_PORT=3306
MARIADB_PORT_3306_TCP_PROTO=tcp
MARIADB_NAME=/servidor_web/mariadb
MARIADB_ENV_MYSQL_DATABASE=mi_basededatos
MARIADB_ENV_MYSQL_USER=usuario
MARIADB_ENV_MYSQL_PASSWORD=asdasd
MARIADB_ENV_MYSQL_ROOT_PASSWORD=asdasd
MARIADB_ENV_GOSU_VERSION=1.14
MARIADB_ENV_MARIADB_MAJOR=10.8
MARIADB_ENV_MARIADB_VERSION=1:10.8.3+maria~jammy
NGINX_VERSION=1.21.6
NJS_VERSION=0.7.3
PKG_RELEASE=1~bullseye
HOME=/root
```

### Uso de las redes bridge definidas por el usuario

Para crear una:

```console
atlas@olympus:~$ docker network create red1
15154211dbeb52ed33a98e45bc9220fa6ec8dd266d72bd8947188a47ac1f3192
```

Si no indicamos direccionamiento, docker lo asigna automáticamente:

```console
atlas@olympus:~$ docker network inspect red1
[
    {
        "Name": "red1",
        "Id": "15154211dbeb52ed33a98e45bc9220fa6ec8dd266d72bd8947188a47ac1f3192",
        "Created": "2022-06-13T20:20:57.861626406+02:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.21.0.0/16",
                    "Gateway": "172.21.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

Conectaremos dos contenedores a esa red:

```console
atlas@olympus:~$ docker run -d --name my-apache-app --network red1 -p 8080:80 httpd:2.4
ff7e218f3527e116d05b77c6e858b0118e49991fb71bc342b502a60a6d533a59
```

Comprobaremos la resolución DNS:

```console
atlas@olympus:~$ docker run -it --name contenedor1 --network red1 debian bash
root@646845be5f23:/# apt update
Get:1 http://security.debian.org/debian-security bullseye-security InRelease [44.1 kB]
Get:2 http://deb.debian.org/debian bullseye InRelease [116 kB]
Get:3 http://deb.debian.org/debian bullseye-updates InRelease [39.4 kB]
Get:4 http://deb.debian.org/debian bullseye/main amd64 Packages [8182 kB]
Get:5 http://security.debian.org/debian-security bullseye-security/main amd64 Packages [154 kB]
Get:6 http://deb.debian.org/debian bullseye-updates/main amd64 Packages [2592 B]
Fetched 8539 kB in 39s (218 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
1 package can be upgraded. Run 'apt list --upgradable' to see it.
root@646845be5f23:/# apt install dnsutils -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  bind9-dnsutils bind9-host bind9-libs libbsd0 libedit2 libfstrm0 libicu67 libjson-c5 liblmdb0 libmaxminddb0 libmd0 libprotobuf-c1 libuv1 libxml2
Suggested packages:
  mmdb-bin
The following NEW packages will be installed:
  bind9-dnsutils bind9-host bind9-libs dnsutils libbsd0 libedit2 libfstrm0 libicu67 libjson-c5 liblmdb0 libmaxminddb0 libmd0 libprotobuf-c1 libuv1 libxml2
0 upgraded, 15 newly installed, 0 to remove and 1 not upgraded.
Need to get 12.2 MB of archives.
After this operation, 42.3 MB of additional disk space will be used.
Selecting previously unselected package libicu67:amd64.
Preparing to unpack .../06-libicu67_67.1-7_amd64.deb ...
Unpacking libicu67:amd64 (67.1-7) ...
Selecting previously unselected package libxml2:amd64.
Preparing to unpack .../07-libxml2_2.9.10+dfsg-6.7+deb11u2_amd64.deb ...
Unpacking libxml2:amd64 (2.9.10+dfsg-6.7+deb11u2) ...
Selecting previously unselected package bind9-libs:amd64.
Preparing to unpack .../08-bind9-libs_1%3a9.16.27-1~deb11u1_amd64.deb ...
Unpacking bind9-libs:amd64 (1:9.16.27-1~deb11u1) ...
Selecting previously unselected package bind9-host.
Preparing to unpack .../09-bind9-host_1%3a9.16.27-1~deb11u1_amd64.deb ...
Unpacking bind9-host (1:9.16.27-1~deb11u1) ...
Selecting previously unselected package libmd0:amd64.
Preparing to unpack .../10-libmd0_1.0.3-3_amd64.deb ...
Unpacking libmd0:amd64 (1.0.3-3) ...
Selecting previously unselected package libbsd0:amd64.
Preparing to unpack .../11-libbsd0_0.11.3-1_amd64.deb ...
Unpacking libbsd0:amd64 (0.11.3-1) ...
Selecting previously unselected package libedit2:amd64.
Preparing to unpack .../12-libedit2_3.1-20191231-2+b1_amd64.deb ...
Unpacking libedit2:amd64 (3.1-20191231-2+b1) ...
Selecting previously unselected package bind9-dnsutils.
Preparing to unpack .../13-bind9-dnsutils_1%3a9.16.27-1~deb11u1_amd64.deb ...
Unpacking bind9-dnsutils (1:9.16.27-1~deb11u1) ...
Selecting previously unselected package dnsutils.
Preparing to unpack .../14-dnsutils_1%3a9.16.27-1~deb11u1_all.deb ...
Unpacking dnsutils (1:9.16.27-1~deb11u1) ...
Setting up liblmdb0:amd64 (0.9.24-1) ...
Setting up libicu67:amd64 (67.1-7) ...
Setting up libmaxminddb0:amd64 (1.5.2-1) ...
Setting up libfstrm0:amd64 (0.6.0-1+b1) ...
Setting up libprotobuf-c1:amd64 (1.3.3-1+b2) ...
Setting up libuv1:amd64 (1.40.0-2) ...
Setting up libmd0:amd64 (1.0.3-3) ...
Setting up libbsd0:amd64 (0.11.3-1) ...
Setting up libjson-c5:amd64 (0.15-2) ...
Setting up libxml2:amd64 (2.9.10+dfsg-6.7+deb11u2) ...
Setting up bind9-libs:amd64 (1:9.16.27-1~deb11u1) ...
Setting up libedit2:amd64 (3.1-20191231-2+b1) ...
Setting up bind9-host (1:9.16.27-1~deb11u1) ...
Setting up bind9-dnsutils (1:9.16.27-1~deb11u1) ...
Setting up dnsutils (1:9.16.27-1~deb11u1) ...
Processing triggers for libc-bin (2.31-13+deb11u3) ...
root@646845be5f23:/# dig my-apache-app

; <<>> DiG 9.16.27-Debian <<>> my-apache-app
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52630
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;my-apache-app.	        IN	A

;; ANSWER SECTION:
my-apache-app.	        600	IN	A	172.21.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Mon Jun 13 19:17:31 UTC 2022
;; MSG SIZE  rcvd: 60
```

Comprobaremos el servidor DNS del contenedor:

```console
root@646845be5f23:/# cat /etc/resolv.conf
nameserver 127.0.0.11
options ndots:0
```

Desde los dos contenedores se pueden resolver los dos nombres:

```console
root@646845be5f23:/# dig contenedor1

; <<>> DiG 9.16.27-Debian <<>> contenedor1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3203
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;contenedor1.		                IN	A

;; ANSWER SECTION:
contenedor1.	        600	IN	A	172.21.0.3

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Mon Jun 13 19:24:38 UTC 2022
;; MSG SIZE  rcvd: 56
```

#### Conectando los contenedores a otras redes

Creamos una red bridge con direccionamiento:

```console
atlas@olympus:~$ docker network create red2 --subnet 192.168.100.0/24 --gateway 192.168.100.1
8090d398ee4247a29a17946a161a6e7746d45c6af098b911f5b0334f56ddd32e
```

Creamos un contenedor conectado a red2 y comprobamos que no tiene conectividad con red1:

```console
atlas@olympus:~$ docker run -it --name contenedor2 --network red2 debian bash
root@bd0dcf22141c:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
11: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:a8:64:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.100.2/24 brd 192.168.100.255 scope global eth0
       valid_lft forever preferred_lft forever
root@bd0dcf22141c:/# ping contenedor1
ping: contenedor1: Name or service not known
```

Conectamos un contenedor a una red:

```console
atlas@olympus:~$ docker network connect red2 contenedor1
atlas@olympus:~$ docker start contenedor1
contenedor1
atlas@olympus:~$ docker start contenedor2
contenedor2
atlas@olympus:~$ docker attach contenedor1
root@4dd5cddfcc4d:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
13: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:15:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.21.0.2/16 brd 172.21.255.255 scope global eth0
       valid_lft forever preferred_lft forever
15: eth1@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:a8:64:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.100.2/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
root@4dd5cddfcc4d:/# ping contenedor2
PING contenedor2 (192.168.100.3) 56(84) bytes of data.
64 bytes from contenedor2.red2 (192.168.100.3): icmp_seq=1 ttl=64 time=0.200 ms
64 bytes from contenedor2.red2 (192.168.100.3): icmp_seq=2 ttl=64 time=0.100 ms
^C
--- contenedor2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.100/0.150/0.200/0.050 ms
```

`contenedor2` tiene conectividad con `contenedor1`, pero no con `my-apache-app` *(porque no está conectado a red2)*:

```console
atlas@olympus:~$ docker attach contenedor2
root@bd0dcf22141c:/# ping contenedor1
PING contenedor1 (192.168.100.2) 56(84) bytes of data.
64 bytes from contenedor1.red2 (192.168.100.2): icmp_seq=1 ttl=64 time=0.185 ms
^C
--- contenedor1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.185/0.185/0.185/0.000 ms
root@bd0dcf22141c:/# ping my-apache-app
ping: my-apache-app: Name or service not known
```

#### Más opciones al trabajar con redes en Docker

Creo una red:

```console
atlas@olympus:~$ docker network create --subnet 192.168.100.0/24 red3
74a6c4d6fa938121164d00f5cbfeda62ffb1b6845b1446679c275c5189a77e40
```

Lanzamos un contenedor en esta red:

```console
atlas@olympus:~$ docker run -it --name contenedor --network red3 \
                                   --ip 192.168.100.10 \
                                   --add-host=testing.example.com:192.168.100.20 \
                                   --dns 8.8.8.8 \
                                   --hostname servidor1 \
                                   debian
root@servidor1:/#
```

`--hostname servidor1` configuró el hostname:

```console
root@servidor1:/# cat /etc/hostname
servidor1
```

`--ip 192.168.100.10` configuró la IP fija:

```console
root@servidor1:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
28: eth0@if29: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:a8:64:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.100.10/24 brd 192.168.100.255 scope global eth0
       valid_lft forever preferred_lft forever
```

`--add-host=testing.example.com:192.168.100.20` configuró un host en `/etc/hosts`:

```console
root@servidor1:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0ip6-localnet
ff00::0ip6-mcastprefix
ff02::1ip6-allnodes
ff02::2ip6-allrouters
192.168.100.20	testing.example.com
192.168.100.10	servidor1
root@servidor1:/# ping testing.example.com
PING testing.example.com (192.168.100.20) 56(84) bytes of data.
^C
--- testing.example.com ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2029ms
```

`--dns 8.8.8.8` configuró el DNS:

```console
root@servidor1:/# cat /etc/resolv.conf
nameserver 127.0.0.11
options ndots:0
```

Este DNS que vemos *(que siempre se crea en las redes bridge de usuario)* hará forward al que nosotros hemos configurado.

### Convertir el uso de la red bridge por defecto por una definida por el usuario

Lanzamos un mongo:

```console
atlas@olympus:~$ docker run -d --name some-mongo mongo
Unable to find image 'mongo:latest' locally
latest: Pulling from library/mongo
d7bfe07ed847: Pull complete
97ef66a8492a: Pull complete
20cec14c8f9e: Pull complete
38c3018eb09a: Pull complete
ccc9e1c2556b: Pull complete
593c62d03532: Pull complete
1a103a446c3f: Pull complete
be887b845d3f: Pull complete
e5543880b183: Pull complete
Digest: sha256:3745209b24062a8ba670878bce47ad9c944682f550044721c4abc5b4daeb863d
Status: Downloaded newer image for mongo:latest
b369ad614dad75e83e45c1bf0faff34ea286d477d4afc7b39b98f1b0199adc5e
```

Lanzamos un lets-chat enlazado con el mongo anterior:

```console
atlas@olympus:~$ docker run --name some-letschat --link some-mongo:mongo -p 8080:8080 -d sdelements/lets-chat
Unable to find image 'sdelements/lets-chat:latest' locally
latest: Pulling from sdelements/lets-chat
6a5a5368e0c2: Pull complete
7b9457ec39de: Pull complete
22cf98377d30: Pull complete
01e6dd0c0aed: Pull complete
eef159360227: Pull complete
a77283727d00: Pull complete
050fb7768585: Pull complete
d2510806c35f: Pull complete
a75c55aa55f6: Pull complete
876c39157780: Pull complete
Digest: sha256:5b923d428176250653530fdac8a9f925043f30c511b77701662d7f8fab74961c
Status: Downloaded newer image for sdelements/lets-chat:latest
f078bbda16971fdf2eb830683e6c72688bb17b6cee07a00976130b419e7d7c7a
```

Probamos que funciona:

![letschatbridgedefecto](https://i.imgur.com/TlPKy8u.jpg)

#### Usando una red bridge de usuario

Creamos la red:

```console
atlas@olympus:~$ docker network create red_letschat
df63456e88c7d16a6bd0af594d4e8b0950f4e26c4b9870d067f7dc8f31dfd7db
```

Lanzamos mongo:

```console
atlas@olympus:~$ docker run -d --network red_letschat --name mongo mongo
3583a31aa1d102886526217cb40f5bccaa088645eab416ad6ea5316dbeb7d1b5
```

Lanzamos lets-chat:

```console
atlas@olympus:~$ docker run --name some-letschat --network red_letschat -p 8080:8080 -d sdelements/lets-chat
1f7157f43c7d3a60136e6908932b6fb1a6e89daf62951eed342e2d4f6dd8292e
```

Probamos que funciona de nuevo:

![letschatbridgeusuario](https://i.imgur.com/OVFhjjQ.jpg)

### Ejemplo 1: Despliegue de la aplicación Guestbook

Creo la red:

```console
atlas@olympus:~$ docker network create red_guestbook
0749760344c197032e1323c4add4a69912b444ab1ff6ce4a799f766d8e0db5e6
```

Lanzo los contenedores:

```console
atlas@olympus:~$ docker run -d --name redis --network red_guestbook redis
Unable to find image 'redis:latest' locally
latest: Pulling from library/redis
42c077c10790: Already exists
a300d83d65f9: Pull complete
ebdc3afaab5c: Pull complete
6ce178c713e4: Pull complete
949f9d8f429f: Pull complete
4076be5e5074: Pull complete
Digest: sha256:cfda0458239615720cc16d6edf6bae7905c31265f218d2033c43cdb40cd59792
Status: Downloaded newer image for redis:latest
96cf2e1a0af36d51cb58848eec71be9bbfaeb9596531fb3bb65b376796917860
atlas@olympus:~$ docker run -d -p 80:5000 --name guestbook --network red_guestbook iesgn/guestbook
Unable to find image 'iesgn/guestbook:latest' locally
latest: Pulling from iesgn/guestbook
0ecb575e629c: Pull complete
7467d1831b69: Pull complete
feab2c490a3c: Pull complete
f15a0f46f8c3: Pull complete
937782447ff6: Pull complete
e78b7aaaab2c: Pull complete
dfce8611166c: Pull complete
3a6aeb6d9625: Pull complete
2b7e1323c92f: Pull complete
4bf029403d35: Pull complete
5e777db1f033: Pull complete
Digest: sha256:a1fc3e0652e324f643d1ebf8ab028d652aa10e13915bcc2c115d5dc9dd0ba7d2
Status: Downloaded newer image for iesgn/guestbook:latest
0d664c2abe850aefdb347c9101b07953409d9e8d33e2b879322effcea40234fe
```

Probamos que funciona:

![guestbookdocker](https://i.imgur.com/9r5mjhY.png)

### Ejemplo 2: Despliegue de la aplicación Temperaturas

Creo la red:

```console
atlas@olympus:~$ docker network create red_temperaturas
d503820887ffa9e2543930675d235f6c56b9f9fdf02524278ec62b7c04e02678
```

Lanzo los contenedores:

```console
atlas@olympus:~$ docker run -d --name temperaturas-backend --network red_temperaturas iesgn/temperaturas_backend
Unable to find image 'iesgn/temperaturas_backend:latest' locally
latest: Pulling from iesgn/temperaturas_backend
0ecb575e629c: Already exists
dd4d4b42fc71: Pull complete
7ce5fbadeda7: Pull complete
54eb73ab1487: Pull complete
Digest: sha256:47bbcb85cfa1e21aecee7cf8fec4fce03197b7e0afd2229fa681116e4d8f422d
Status: Downloaded newer image for iesgn/temperaturas_backend:latest
ef1fbd2d31ac6c031f6bdb374983d6f7552ea8f591d4d062c57fb6ac4be43e5b
atlas@olympus:~$ docker run -d -p 80:3000 --name temperaturas-frontend --network red_temperaturas iesgn/temperaturas_frontend
Unable to find image 'iesgn/temperaturas_frontend:latest' locally
latest: Pulling from iesgn/temperaturas_frontend
0ecb575e629c: Already exists
1b0ea791d129: Pull complete
18ffb3e704a1: Pull complete
d724eea63276: Pull complete
Digest: sha256:7a2b7b66f830e0905d2b0b7faf33a5bea3da7c12c8ea75abdc034603307c7677
Status: Downloaded newer image for iesgn/temperaturas_frontend:latest
8ffd4cf58fa19fb7f39ed690b7a724b0c2e364e66bba1359ae3ba5212e48f4ae
```

Probamos que funciona:

![temperaturasdocker](https://i.imgur.com/dnB3nkQ.png)

### Ejemplo 3: Despliegue de Wordpress + MariaDB

Creo la red:

```console
atlas@olympus:~$ docker network create red_wp
80c955b07b17dcf9eda18567b0b65cba6344bf3ac45beba35eff131e77630abf
```

Lanzo los contenedores:

```console
atlas@olympus:~$ docker run -d --name servidor_mysql \
                --network red_wp \
                -v /opt/mysql_wp:/var/lib/mysql \
                -e MYSQL_DATABASE=bd_wp \
                -e MYSQL_USER=user_wp \
                -e MYSQL_PASSWORD=asdasd \
                -e MYSQL_ROOT_PASSWORD=asdasd \
                mariadb
3d892aa3847a19b4fc877dfa2dd085e7182d07f01ccc9be42fd6aa731e20255e
atlas@olympus:~$ docker run -d --name servidor_wp \
                --network red_wp \
                -v /opt/wordpress:/var/www/html/wp-content \
                -e WORDPRESS_DB_HOST=servidor_mysql \
                -e WORDPRESS_DB_USER=user_wp \
                -e WORDPRESS_DB_PASSWORD=asdasd \
                -e WORDPRESS_DB_NAME=bd_wp \
                -p 80:80 \
                wordpress
Unable to find image 'wordpress:latest' locally
latest: Pulling from library/wordpress
42c077c10790: Already exists
8934009a9160: Already exists
5357ac116991: Already exists
54ae63894b5a: Already exists
772088206f85: Already exists
3b81c5474649: Already exists
c62a528527ae: Already exists
344c792709d4: Already exists
ef9638273c35: Already exists
4ec70638e42f: Already exists
5a0f65f5bb3b: Already exists
55578a5ce5e8: Already exists
f1dd85b4dfa5: Already exists
35dc259d9d51: Pull complete
48147606c534: Pull complete
c88c969d9899: Pull complete
e08b6500594a: Pull complete
027690aff84d: Pull complete
8770aef363f4: Pull complete
336e4cd7d982: Pull complete
8e0e84b1add6: Pull complete
Digest: sha256:92a0f6f7e92eab575659ff6c6ee3d1abb56752977d1d729de5da1a876f9f1f42
Status: Downloaded newer image for wordpress:latest
ec1b9fac1479fde88682d5071b86f198b69a9442064b29bc3ac5770ed1a07b33
```

Muestro que están activos:

```console
atlas@olympus:~$ docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                NAMES
ec1b9fac1479   wordpress   "docker-entrypoint.s…"   57 seconds ago   Up 55 seconds   0.0.0.0:80->80/tcp   servidor_wp
3d892aa3847a   mariadb     "docker-entrypoint.s…"   2 minutes ago    Up 2 minutes    3306/tcp             servidor_mysql
```

Probamos que funciona *(haré la instalación y lo mostraré directamente funcionando)*:

![wpdocker](https://i.imgur.com/bJsyuQo.png)

### Ejemplo 4: Despliegue de Tomcat + Nginx

Creo la red:

```console
atlas@olympus:~$ docker network create red_tomcat
891d29f1d2054410e0c43654924fe8400584ea1d2b411262a9d4085182e60234
```

Creo el directorio tomcat y entro:

```console
atlas@olympus:~$ mkdir tomcat
atlas@olympus:~$ cd tomcat
atlas@olympus:~/tomcat$
```

Descargo en él los 2 ficheros que necesito y muestro que los tengo:

```console
atlas@olympus:~/tomcat$ wget https://raw.githubusercontent.com/iesgn/curso_docker_2021/main/ejemplos/sesion4/ejemplo4/default.conf
--2022-06-14 20:35:44--  https://raw.githubusercontent.com/iesgn/curso_docker_2021/main/ejemplos/sesion4/ejemplo4/default.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.108.133, 185.199.111.133, 185.199.109.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 387 [text/plain]
Saving to: ‘default.conf’

default.conf                                100%[=========================================================================================>]     387  --.-KB/s    in 0s

2022-06-14 20:35:44 (15.6 MB/s) - ‘default.conf’ saved [387/387]
atlas@olympus:~/tomcat$ wget https://github.com/iesgn/curso_docker_2021/raw/main/ejemplos/sesion4/ejemplo4/sample.war
--2022-06-14 20:38:59--  https://github.com/iesgn/curso_docker_2021/raw/main/ejemplos/sesion4/ejemplo4/sample.war
Resolving github.com (github.com)... 140.82.121.4
Connecting to github.com (github.com)|140.82.121.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/iesgn/curso_docker_2021/main/ejemplos/sesion4/ejemplo4/sample.war [following]
--2022-06-14 20:38:59--  https://raw.githubusercontent.com/iesgn/curso_docker_2021/main/ejemplos/sesion4/ejemplo4/sample.war
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.109.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4606 (4.5K) [application/octet-stream]
Saving to: ‘sample.war’

sample.war                                  100%[=========================================================================================>]   4.50K  --.-KB/s    in 0.002s

2022-06-14 20:38:59 (2.21 MB/s) - ‘sample.war’ saved [4606/4606]
atlas@olympus:~/tomcat$ ls -la
total 20
drwxr-xr-x  2 atlas atlas 4096 Jun 14 20:38 .
drwxr-xr-x 42 atlas atlas 4096 Jun 14 20:32 ..
-rw-r--r--  1 atlas atlas  387 Jun 14 20:35 default.conf
-rw-r--r--  1 atlas atlas 4606 Jun 14 20:38 sample.war
```

Lanzo el contenedor de Tomcat:

```console
atlas@olympus:~/tomcat$ docker run -d --name aplicacionjava \
                --network red_tomcat \
                -v /home/atlas/tomcat/sample.war:/usr/local/tomcat/webapps/sample.war:ro \
                tomcat:9.0
Unable to find image 'tomcat:9.0' locally
9.0: Pulling from library/tomcat
e756f3fdd6a3: Already exists
bf168a674899: Pull complete
e604223835cc: Pull complete
6d5c91c4cd86: Pull complete
5e20d165240e: Pull complete
1334d60df9a8: Pull complete
16c2728dcd90: Pull complete
05288798d23d: Pull complete
3bbce73846a1: Pull complete
58e753cca0c7: Pull complete
Digest: sha256:8558e9b51d8aa9f08d6611c54a832b1be64636e713532729fca44774d6a0765d
Status: Downloaded newer image for tomcat:9.0
bd9c22e54e6e7b96e27ab502c0db185ca24ea87635d222c8e16aed84e92dc6c6
```

Lanzo el contenedor de Nginx:

```console
atlas@olympus:~/tomcat$ docker run -d --name proxy \
                -p 80:80 \
                --network red_tomcat \
                -v /home/atlas/tomcat/default.conf:/etc/nginx/conf.d/default.conf:ro \
                nginx
a62ecf82ea16924e06a49bbedd7a32857a2ebebb04becde4b0e57ed8d4f31751
```

Probamos que funciona:

![tomcatdocker](https://i.imgur.com/9INKo2S.png)

### Ejercicios redes

Creo `red1`:

```console
atlas@olympus:~$ docker network create red1 --subnet 172.28.0.0/16 --gateway 172.28.0.1
83eb74c0d72db336bd40fd2c48cdd6583b8b232b304ecc858fde6e5d05432709
```

Creo `red2`:

```console
atlas@olympus:~$ docker network create red2
501db55f3c489ba353ac40105eea56c5053707f04199af73dc47b38f317e677d
```

Lanzo el primer contenedor e instalo ping:

```console
atlas@olympus:~$ docker run -it --name u1 --network red1 --ip 172.28.0.10 --hostname host1 ubuntu:20.04
root@host1:/# apt update
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:4 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [27.5 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:6 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [880 kB]
Get:7 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [1931 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:9 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
Get:10 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [1286 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [2364 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [1365 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1169 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [30.2 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [27.1 kB]
Get:18 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [53.8 kB]
Fetched 22.6 MB in 9s (2566 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
7 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@host1:/# apt install inetutils-ping
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  libidn11 netbase
The following NEW packages will be installed:
  inetutils-ping libidn11 netbase
0 upgraded, 3 newly installed, 0 to remove and 7 not upgraded.
Need to get 120 kB of archives.
After this operation, 657 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu focal/main amd64 netbase all 6.1 [13.1 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal/main amd64 libidn11 amd64 1.33-2.2ubuntu2 [46.2 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 inetutils-ping amd64 2:1.9.4-11ubuntu0.1 [60.7 kB]
Fetched 120 kB in 1s (113 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package netbase.
(Reading database ... 4126 files and directories currently installed.)
Preparing to unpack .../archives/netbase_6.1_all.deb ...
Unpacking netbase (6.1) ...
Selecting previously unselected package libidn11:amd64.
Preparing to unpack .../libidn11_1.33-2.2ubuntu2_amd64.deb ...
Unpacking libidn11:amd64 (1.33-2.2ubuntu2) ...
Selecting previously unselected package inetutils-ping.
Preparing to unpack .../inetutils-ping_2%3a1.9.4-11ubuntu0.1_amd64.deb ...
Unpacking inetutils-ping (2:1.9.4-11ubuntu0.1) ...
Setting up libidn11:amd64 (1.33-2.2ubuntu2) ...
Setting up netbase (6.1) ...
Setting up inetutils-ping (2:1.9.4-11ubuntu0.1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
```

Muestro la configuración de red de `u1`:

[![asciicast](https://www.dataquest.io/wp-content/uploads/2019/07/command-line-courses-dataquest-1000x520-1-1.gif)](https://asciinema.org/a/OYKnsP6PL81gtiyzhEVispzkx)


<a href="https://asciinema.org/a/OYKnsP6PL81gtiyzhEVispzkx"><img src="https://asciinema.org/a/14.png" width="836"/></a>



Lanzo el segundo contenedor e instalo ping:

```console

```











