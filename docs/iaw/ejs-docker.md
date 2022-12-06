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

Mostrar la configuración de red de `u1`:

```console
docker inspect u1
```

![redu1](https://i.imgur.com/ibRoGod.png)

Lanzo el segundo contenedor e instalo ping:

```console
atlas@olympus:~$ docker run -it --name u2 --network red2 --hostname host2 ubuntu:20.04
root@host2:/# apt update
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:6 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:7 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [880 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]
Get:9 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
Get:10 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [1931 kB]
Get:11 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [27.5 kB]
Get:12 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [1286 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1169 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [1365 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [2364 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [30.2 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [53.8 kB]
Get:18 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [27.1 kB]
Fetched 22.6 MB in 25s (905 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
7 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@host2:/# apt install inetutils-ping
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
Fetched 120 kB in 0s (3242 kB/s)
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

Mostrar la configuración de red de `u2`:

```console
docker inspect u2
```

![redu2](https://i.postimg.cc/jjZS6Xmc/red2u2.png)

Compruebo que el ping IP `u1` &rarr; `u2` no funciona:

![nopingipu1u2](https://i.imgur.com/S1G7qRP.png)

Compruebo que el ping por nombre `u1` &rarr; `u2` no funciona:

![nopingnombreu1u2](https://i.imgur.com/B3GsOG9.png)

Compruebo que el ping IP `u2` &rarr; `u1` no funciona:

![nopingipu2u1](https://i.imgur.com/PIng3pl.png)

Compruebo que el ping por nombre `u2` &rarr; `u1` no funciona:

![nopingnombreu2u1](https://i.imgur.com/C2YRHTm.png)

Conectamos ahora `u1` a `red2`:

![u1conectadored2](https://i.imgur.com/kw5FxLr.png)

Comprobamos que ahora el ping IP/nombre `u1` &rarr; `u2` funciona:

![u1u2funciona](https://i.imgur.com/an6k4cm.png)

## Escenarios multicontenedor

### Instalación de docker-compose

```console
sudo apt install docker-compose
```

### El fichero docker-compose.yml

`docker-compose` se ejecuta en el mismo directorio donde esté el `docker-compose.yml`, por lo que acabaremos teniendo un directorio por cada `docker-compose.yml`.

Por ejemplo, para `Let’s Chat`, podríamos tener este `docker-compose.yml` dentro de un directorio:

```console
version: '3.1'
services:
  app:
    container_name: letschat
    image: sdelements/lets-chat
    restart: always
    environment:
      LCB_DATABASE_URI: mongodb://mongo/letschat
    ports:
      - 80:8080
    depends_on:
      - db
  db:
    container_name: mongo
    image: mongo
    restart: always
    volumes:
      - /opt/mongo:/data/db
```

Cada `service` crea un contenedor.

`depends_on` obliga a que X contenedor no se inicie hasta que otro esté iniciado.

Por cada escenario `docker-compose` se crea un bridge de usuario. Existirá resolución DNS para nombre de contenedor y de servicio.

### El comando docker-compose

Creo la estructura de directorios `/home/atlas/docker/letschat` con el siguiente `docker-compose.yml` dentro:

```console
version: '3.1'
services:
  app:
    container_name: letschat
    image: sdelements/lets-chat
    restart: always
    environment:
      LCB_DATABASE_URI: mongodb://mongo/letschat
    ports:
      - 80:8080
    depends_on:
      - db
  db:
    container_name: mongo
    image: mongo
    restart: always
    volumes:
      - /opt/mongo:/data/db
```

Lanzo el escenario:

```console
atlas@olympus:~/docker/letschat$ docker-compose up -d
Creating mongo ... done
Creating letschat ... done
```

Comprobamos que los contenedores se están ejecutando:

```console
atlas@olympus:~/docker/letschat$ docker-compose ps
  Name               Command             State               Ports
-------------------------------------------------------------------------------
letschat   npm start                     Up      5222/tcp, 0.0.0.0:80->8080/tcp
mongo      docker-entrypoint.sh mongod   Up      27017/tcp
```

Comprobamos que funciona:

![letschatcompose](https://i.imgur.com/WVRmAXI.jpg)

Para eliminar el escenario:

```console
atlas@olympus:~/docker/letschat$ docker-compose down
Stopping letschat ... done
Stopping mongo    ... done
Removing letschat ... done
Removing mongo    ... done
Removing network letschat_default
```

### Almacenamiento con docker-compose

#### Volúmenes docker-compose

Ejemplo:

```console
version: '3.1'
services:
  db:
    container_name: contenedor_mariadb
    image: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: asdasd
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:
```

Lanzo el escenario:

```console
atlas@olympus:~/docker/volumenes$ docker-compose up -d
Creating network "volumenes_default" with the default driver
Creating volume "volumenes_mariadb_data" with default driver
Creating contenedor_mariadb ... done
```

Compruebo que está activo:

```console
atlas@olympus:~/docker/volumenes$ docker-compose ps
       Name                     Command              State    Ports
---------------------------------------------------------------------
contenedor_mariadb   docker-entrypoint.sh mariadbd   Up      3306/tcp
```

Compruebo que se ha creado el volumen:

```console
atlas@olympus:~/docker/volumenes$ docker volume ls
DRIVER    VOLUME NAME
local     3f39653a3a30f881672c01433725daeb04ab6ba632a723b1af18b093f50e06ae
local     04e463e496d920db4b021cd270aa770d27abaaf32a08d0d035d1fe03356f87b3
local     7df3bfbe82aaf633a16cafb30628c3d0384d48404851b99dc892cb337d530950
local     7e6492fdc4515b59e4d3e86d9c69f31b6e771d78e3c8231dbb0a3eed24bed33c
local     07c295fb652fc3ec7f0f51a3df2d13849aac8795b44e4b55e22e371dea84022d
local     19b03e7f9608f78aac80ee1284c4fa4c5f78afb723499cc664f7753971639753
local     65eb5fc7fc9f541f1dd35effc2e456e615f6ef51670ef07a9971d2278c9dd4ee
local     76c389c617b86588b5950575e9bfbb3f9a17316c491880f30621d3e3bcbe251f
local     256e88e1d29e428176b13909592ca4f124845226dec90aae73f9ddfd26021c9c
local     582b8ad89272ee7607a9e56dfaeae28c1eba9a7263a8052d64e82e6361b4ef1c
local     972306d6e9955703c733bd57d9e270d953e8a028760fd6d3af13c2085af8d1f8
local     d13dcfcda6c5bcbb2b75fdce9f7b74136b88d82d8d4c2e08e6b957840273b0b4
local     da9451ceb333f905d8a8fbe4e764ab2a057139f1641a024ffae408b81cfb72d7
local     datos_compartidos
local     e6c217e966db3c2491c2b6989789f6c7488aaa1222313415722e3d06aaedca03
local     f108fc0c5812145adaa6d4987a0afd9748876ff8bd7b5520e916eb530a3d35c3
local     f9783771ca7c6480508aab09a722d746803e0955cd6b0e8cadcebfd7f1aff44a
local     nextcloud
local     volumen_web
local     volumenes_mariadb_data
local     wwwroot
```

Compruebo que el montaje del volumen se ha realizado en el contenedor:

```console
docker inspect contenedor_mariadb
```

```console
        "Mounts": [
            {
                "Type": "volume",
                "Name": "volumenes_mariadb_data",
                "Source": "/var/lib/docker/volumes/volumenes_mariadb_data/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "rw",
                "RW": true,
                "Propagation": ""
            }
        ],
```

Elimino escenario + posibles volúmenes:

```console
atlas@olympus:~/docker/volumenes$ docker-compose down -v
Stopping contenedor_mariadb ... done
Removing contenedor_mariadb ... done
Removing network volumenes_default
Removing volume volumenes_mariadb_data
```

#### Bind mount docker-compose

Ejemplo:

```console
version: '3.1'
services:
  db:
    container_name: contenedor_mariadb
    image: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: asdasd
    volumes:
      - ./data:/var/lib/mysql
```

Lanzo el escenario

```console
atlas@olympus:~/docker/bindmount$ docker-compose up -d
Creating network "bindmount_default" with the default driver
Creating contenedor_mariadb ... done
```

Compruebo que se ha creado el directorio `data`:

```console
atlas@olympus:~/docker/bindmount$ cd data
atlas@olympus:~/docker/bindmount/data$ ls -la
total 139260
drwxr-xr-x 5 systemd-coredump systemd-coredump      4096 Jun 15 21:57 .
drwxr-xr-x 3 atlas            atlas                 4096 Jun 15 21:57 ..
-rw-rw---- 1 systemd-coredump systemd-coredump  16719872 Jun 15 21:57 aria_log.00000001
-rw-rw---- 1 systemd-coredump systemd-coredump        52 Jun 15 21:57 aria_log_control
-rw-rw---- 1 systemd-coredump systemd-coredump         9 Jun 15 21:57 ddl_recovery.log
-rw-rw---- 1 systemd-coredump systemd-coredump       868 Jun 15 21:57 ib_buffer_pool
-rw-rw---- 1 systemd-coredump systemd-coredump  12582912 Jun 15 21:57 ibdata1
-rw-rw---- 1 systemd-coredump systemd-coredump 100663296 Jun 15 21:57 ib_logfile0
-rw-rw---- 1 systemd-coredump systemd-coredump  12582912 Jun 15 21:57 ibtmp1
-rw-rw---- 1 systemd-coredump systemd-coredump         0 Jun 15 21:57 multi-master.info
drwx------ 2 systemd-coredump systemd-coredump      4096 Jun 15 21:57 mysql
-rw-r--r-- 1 systemd-coredump systemd-coredump        14 Jun 15 21:57 mysql_upgrade_info
drwx------ 2 systemd-coredump systemd-coredump      4096 Jun 15 21:57 performance_schema
drwx------ 2 systemd-coredump systemd-coredump     12288 Jun 15 21:57 sys
```

### Redes con docker-compose

Ejemplo:

```console
version: '3.1'
services:
  app:
    container_name: servidor_web
    image: httpd:2.4
    restart: always
    ports:
      - 8080:80
    networks:
      red_web:
        ipv4_address: 192.168.10.10
      red_interna:
        ipv4_address: 192.168.20.10
    hostname: servidor_web

  db:
    container_name: servidor_mariadb
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: asdasd
    restart: always
    networks:
      red_interna:
        ipv4_address: 192.168.20.20
    hostname: servidor_mariadb
networks:
    red_web:
        ipam:
            config:
              - subnet: 192.168.10.0/24
    red_interna:
        ipam:
            config:
              - subnet: 192.168.20.0/24
```

Lanzo el escenario:

```console
atlas@olympus:~/docker/redes$ docker-compose up -d
Creating network "redes_red_web" with the default driver
Creating network "redes_red_interna" with the default driver
Creating servidor_web     ... done
Creating servidor_mariadb ... done
```

Compruebo que los contenedores se están ejecutando:

```console
atlas@olympus:~/docker/redes$ docker-compose ps
      Name                    Command              State          Ports
-------------------------------------------------------------------------------
servidor_mariadb   docker-entrypoint.sh mariadbd   Up      3306/tcp
servidor_web       httpd-foreground                Up      0.0.0.0:8080->80/tcp
```

Accedo a `servidor_web` e instalo paquetes para comprobaciones:

```console
atlas@olympus:~/docker/redes$ docker-compose exec app bash
root@servidor_web:/usr/local/apache2# apt-get update && apt-get install -y inetutils-ping iproute2 dnsutils
Get:1 http://deb.debian.org/debian bullseye InRelease [116 kB]
Get:2 http://security.debian.org/debian-security bullseye-security InRelease [44.1 kB]
Get:3 http://deb.debian.org/debian bullseye-updates InRelease [39.4 kB]
Get:4 http://deb.debian.org/debian bullseye/main amd64 Packages [8182 kB]
Get:5 http://security.debian.org/debian-security bullseye-security/main amd64 Packages [154 kB]
Get:6 http://deb.debian.org/debian bullseye-updates/main amd64 Packages [2592 B]
Fetched 8539 kB in 4s (2245 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  bind9-dnsutils bind9-host bind9-libs libatm1 libbpf0 libbsd0 libcap2 libcap2-bin libedit2 libelf1 libfstrm0 libjson-c5 liblmdb0 libmaxminddb0 libmd0 libmnl0 libpam-cap
  libprotobuf-c1 libuv1 libxtables12 netbase
Suggested packages:
  iproute2-doc mmdb-bin
The following NEW packages will be installed:
  bind9-dnsutils bind9-host bind9-libs dnsutils inetutils-ping iproute2 libatm1 libbpf0 libbsd0 libcap2 libcap2-bin libedit2 libelf1 libfstrm0 libjson-c5 liblmdb0
  libmaxminddb0 libmd0 libmnl0 libpam-cap libprotobuf-c1 libuv1 libxtables12 netbase
0 upgraded, 24 newly installed, 0 to remove and 1 not upgraded.
Need to get 4565 kB of archives.
After this operation, 11.6 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 libelf1 amd64 0.183-1 [165 kB]
Get:2 http://deb.debian.org/debian bullseye/main amd64 libbpf0 amd64 1:0.3-2 [98.3 kB]
Get:3 http://deb.debian.org/debian bullseye/main amd64 libmd0 amd64 1.0.3-3 [28.0 kB]
Get:4 http://deb.debian.org/debian bullseye/main amd64 libbsd0 amd64 0.11.3-1 [108 kB]
Get:5 http://deb.debian.org/debian bullseye/main amd64 libcap2 amd64 1:2.44-1 [23.6 kB]
Get:6 http://deb.debian.org/debian bullseye/main amd64 libmnl0 amd64 1.0.4-3 [12.5 kB]
Get:7 http://deb.debian.org/debian bullseye/main amd64 libxtables12 amd64 1.8.7-1 [45.1 kB]
Get:8 http://deb.debian.org/debian bullseye/main amd64 libcap2-bin amd64 1:2.44-1 [32.6 kB]
Get:9 http://deb.debian.org/debian bullseye/main amd64 iproute2 amd64 5.10.0-4 [930 kB]
Get:10 http://deb.debian.org/debian bullseye/main amd64 netbase all 6.3 [19.9 kB]
Get:11 http://deb.debian.org/debian bullseye/main amd64 libfstrm0 amd64 0.6.0-1+b1 [21.5 kB]
Get:12 http://deb.debian.org/debian bullseye/main amd64 libjson-c5 amd64 0.15-2 [42.8 kB]
Get:13 http://deb.debian.org/debian bullseye/main amd64 liblmdb0 amd64 0.9.24-1 [45.0 kB]
Get:14 http://deb.debian.org/debian bullseye/main amd64 libmaxminddb0 amd64 1.5.2-1 [29.8 kB]
Get:15 http://deb.debian.org/debian bullseye/main amd64 libprotobuf-c1 amd64 1.3.3-1+b2 [27.0 kB]
Get:16 http://deb.debian.org/debian bullseye/main amd64 libuv1 amd64 1.40.0-2 [132 kB]
Get:17 http://deb.debian.org/debian bullseye/main amd64 bind9-libs amd64 1:9.16.27-1~deb11u1 [1413 kB]
Get:18 http://deb.debian.org/debian bullseye/main amd64 bind9-host amd64 1:9.16.27-1~deb11u1 [302 kB]
Get:19 http://deb.debian.org/debian bullseye/main amd64 libedit2 amd64 3.1-20191231-2+b1 [96.7 kB]
Get:20 http://deb.debian.org/debian bullseye/main amd64 bind9-dnsutils amd64 1:9.16.27-1~deb11u1 [398 kB]
Get:21 http://deb.debian.org/debian bullseye/main amd64 dnsutils all 1:9.16.27-1~deb11u1 [262 kB]
Get:22 http://deb.debian.org/debian bullseye/main amd64 inetutils-ping amd64 2:2.0-1 [245 kB]
Get:23 http://deb.debian.org/debian bullseye/main amd64 libatm1 amd64 1:2.5.1-4 [71.3 kB]
Get:24 http://deb.debian.org/debian bullseye/main amd64 libpam-cap amd64 1:2.44-1 [15.4 kB]
Fetched 4565 kB in 2s (2995 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libelf1:amd64.
(Reading database ... 6833 files and directories currently installed.)
Preparing to unpack .../00-libelf1_0.183-1_amd64.deb ...
Unpacking libelf1:amd64 (0.183-1) ...
Selecting previously unselected package libbpf0:amd64.
Preparing to unpack .../01-libbpf0_1%3a0.3-2_amd64.deb ...
Unpacking libbpf0:amd64 (1:0.3-2) ...
Selecting previously unselected package libmd0:amd64.
Preparing to unpack .../02-libmd0_1.0.3-3_amd64.deb ...
Unpacking libmd0:amd64 (1.0.3-3) ...
Selecting previously unselected package libbsd0:amd64.
Preparing to unpack .../03-libbsd0_0.11.3-1_amd64.deb ...
Unpacking libbsd0:amd64 (0.11.3-1) ...
Selecting previously unselected package libcap2:amd64.
Preparing to unpack .../04-libcap2_1%3a2.44-1_amd64.deb ...
Unpacking libcap2:amd64 (1:2.44-1) ...
Selecting previously unselected package libmnl0:amd64.
Preparing to unpack .../05-libmnl0_1.0.4-3_amd64.deb ...
Unpacking libmnl0:amd64 (1.0.4-3) ...
Selecting previously unselected package libxtables12:amd64.
Preparing to unpack .../06-libxtables12_1.8.7-1_amd64.deb ...
Unpacking libxtables12:amd64 (1.8.7-1) ...
Selecting previously unselected package libcap2-bin.
Preparing to unpack .../07-libcap2-bin_1%3a2.44-1_amd64.deb ...
Unpacking libcap2-bin (1:2.44-1) ...
Selecting previously unselected package iproute2.
Preparing to unpack .../08-iproute2_5.10.0-4_amd64.deb ...
Unpacking iproute2 (5.10.0-4) ...
Selecting previously unselected package netbase.
Preparing to unpack .../09-netbase_6.3_all.deb ...
Unpacking netbase (6.3) ...
Selecting previously unselected package libfstrm0:amd64.
Preparing to unpack .../10-libfstrm0_0.6.0-1+b1_amd64.deb ...
Unpacking libfstrm0:amd64 (0.6.0-1+b1) ...
Selecting previously unselected package libjson-c5:amd64.
Preparing to unpack .../11-libjson-c5_0.15-2_amd64.deb ...
Unpacking libjson-c5:amd64 (0.15-2) ...
Selecting previously unselected package liblmdb0:amd64.
Preparing to unpack .../12-liblmdb0_0.9.24-1_amd64.deb ...
Unpacking liblmdb0:amd64 (0.9.24-1) ...
Selecting previously unselected package libmaxminddb0:amd64.
Preparing to unpack .../13-libmaxminddb0_1.5.2-1_amd64.deb ...
Unpacking libmaxminddb0:amd64 (1.5.2-1) ...
Selecting previously unselected package libprotobuf-c1:amd64.
Preparing to unpack .../14-libprotobuf-c1_1.3.3-1+b2_amd64.deb ...
Unpacking libprotobuf-c1:amd64 (1.3.3-1+b2) ...
Selecting previously unselected package libuv1:amd64.
Preparing to unpack .../15-libuv1_1.40.0-2_amd64.deb ...
Unpacking libuv1:amd64 (1.40.0-2) ...
Selecting previously unselected package bind9-libs:amd64.
Preparing to unpack .../16-bind9-libs_1%3a9.16.27-1~deb11u1_amd64.deb ...
Unpacking bind9-libs:amd64 (1:9.16.27-1~deb11u1) ...
Selecting previously unselected package bind9-host.
Preparing to unpack .../17-bind9-host_1%3a9.16.27-1~deb11u1_amd64.deb ...
Unpacking bind9-host (1:9.16.27-1~deb11u1) ...
Selecting previously unselected package libedit2:amd64.
Preparing to unpack .../18-libedit2_3.1-20191231-2+b1_amd64.deb ...
Unpacking libedit2:amd64 (3.1-20191231-2+b1) ...
Selecting previously unselected package bind9-dnsutils.
Preparing to unpack .../19-bind9-dnsutils_1%3a9.16.27-1~deb11u1_amd64.deb ...
Unpacking bind9-dnsutils (1:9.16.27-1~deb11u1) ...
Selecting previously unselected package dnsutils.
Preparing to unpack .../20-dnsutils_1%3a9.16.27-1~deb11u1_all.deb ...
Unpacking dnsutils (1:9.16.27-1~deb11u1) ...
Selecting previously unselected package inetutils-ping.
Preparing to unpack .../21-inetutils-ping_2%3a2.0-1_amd64.deb ...
Unpacking inetutils-ping (2:2.0-1) ...
Selecting previously unselected package libatm1:amd64.
Preparing to unpack .../22-libatm1_1%3a2.5.1-4_amd64.deb ...
Unpacking libatm1:amd64 (1:2.5.1-4) ...
Selecting previously unselected package libpam-cap:amd64.
Preparing to unpack .../23-libpam-cap_1%3a2.44-1_amd64.deb ...
Unpacking libpam-cap:amd64 (1:2.44-1) ...
Setting up liblmdb0:amd64 (0.9.24-1) ...
Setting up libmaxminddb0:amd64 (1.5.2-1) ...
Setting up libfstrm0:amd64 (0.6.0-1+b1) ...
Setting up libatm1:amd64 (1:2.5.1-4) ...
Setting up libprotobuf-c1:amd64 (1.3.3-1+b2) ...
Setting up libcap2:amd64 (1:2.44-1) ...
Setting up libcap2-bin (1:2.44-1) ...
Setting up libuv1:amd64 (1.40.0-2) ...
Setting up libmnl0:amd64 (1.0.4-3) ...
Setting up libxtables12:amd64 (1.8.7-1) ...
Setting up libmd0:amd64 (1.0.3-3) ...
Setting up netbase (6.3) ...
Setting up libbsd0:amd64 (0.11.3-1) ...
Setting up libelf1:amd64 (0.183-1) ...
Setting up libpam-cap:amd64 (1:2.44-1) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 78.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.32.1 /usr/local/share/perl/5.32.1 /usr/lib/x86_64-linux-gnu/perl5/5.32 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl-base /usr/lib/x86_64-linux-gnu/perl/5.32 /usr/share/perl/5.32 /usr/local/lib/site_perl) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up libjson-c5:amd64 (0.15-2) ...
Setting up bind9-libs:amd64 (1:9.16.27-1~deb11u1) ...
Setting up libedit2:amd64 (3.1-20191231-2+b1) ...
Setting up inetutils-ping (2:2.0-1) ...
Setting up libbpf0:amd64 (1:0.3-2) ...
Setting up bind9-host (1:9.16.27-1~deb11u1) ...
Setting up iproute2 (5.10.0-4) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 78.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.32.1 /usr/local/share/perl/5.32.1 /usr/lib/x86_64-linux-gnu/perl5/5.32 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl-base /usr/lib/x86_64-linux-gnu/perl/5.32 /usr/share/perl/5.32 /usr/local/lib/site_perl) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up bind9-dnsutils (1:9.16.27-1~deb11u1) ...
Setting up dnsutils (1:9.16.27-1~deb11u1) ...
Processing triggers for libc-bin (2.31-13+deb11u3) ...
```

Compruebo que el hostname es correcto:

```console
root@servidor_web:/usr/local/apache2# cat /etc/hostname
servidor_web
```

Compruebo que `servidor_web` está conectado a las dos redes:

```console
root@servidor_web:/usr/local/apache2# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
30: eth0@if31: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:a8:14:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.20.10/24 brd 192.168.20.255 scope global eth0
       valid_lft forever preferred_lft forever
32: eth1@if33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:a8:0a:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.10.10/24 brd 192.168.10.255 scope global eth1
       valid_lft forever preferred_lft forever
```

Compruebo que hay resolución DNS al nombre de servicio/contenedor `servidor_web` &rarr; `servidor_mariadb`:

```console
root@servidor_web:/usr/local/apache2# dig servidor_mariadb

; <<>> DiG 9.16.27-Debian <<>> servidor_mariadb
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38120
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;servidor_mariadb.	        IN	A

;; ANSWER SECTION:
servidor_mariadb.	600	IN	A	192.168.20.20

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Wed Jun 15 21:08:40 UTC 2022
;; MSG SIZE  rcvd: 66

root@servidor_web:/usr/local/apache2# dig db

; <<>> DiG 9.16.27-Debian <<>> db
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32469
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;db.		                        IN	A

;; ANSWER SECTION:
db.		                600	IN	A	192.168.20.20

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Wed Jun 15 21:09:29 UTC 2022
;; MSG SIZE  rcvd: 38
```

Compruebo que hay ping `servidor_web` &rarr; `servidor_mariadb`:

```console
root@servidor_web:/usr/local/apache2# ping servidor_mariadb
PING servidor_mariadb (192.168.20.20): 56 data bytes
64 bytes from 192.168.20.20: icmp_seq=0 ttl=64 time=0.222 ms
64 bytes from 192.168.20.20: icmp_seq=1 ttl=64 time=0.132 ms
^C--- servidor_mariadb ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.132/0.177/0.222/0.045 ms
```

### Ejemplo 1: Despliegue de la aplicación guestbook

Creo el directorio y entro:

```console
atlas@olympus:~/docker$ mkdir guestbook
atlas@olympus:~/docker$ cd guestbook
atlas@olympus:~/docker/guestbook$
```

Creo el siguiente `docker-compose.yml`:

```console
version: '3.1'
services:
  app:
    container_name: guestbook
    image: iesgn/guestbook
    restart: always
    ports:
      - 80:5000
  db:
    container_name: redis
    image: redis
    restart: always
```

Lanzo el escenario:

```console
atlas@olympus:~/docker/guestbook$ docker-compose up -d
Creating network "guestbook_default" with the default driver
Creating redis     ... done
Creating guestbook ... done
```

Listo los contenedores:

```console
atlas@olympus:~/docker/guestbook$ docker-compose ps
  Name                 Command               State          Ports
-------------------------------------------------------------------------
guestbook   python3 app.py                   Up      0.0.0.0:80->5000/tcp
redis       docker-entrypoint.sh redis ...   Up      6379/tcp
```

Pruebo que funciona:

![guestbookcompose](https://i.imgur.com/WDrh3W2.png)

Paro los contenedores:

```console
atlas@olympus:~/docker/guestbook$ docker-compose stop
Stopping guestbook ... done
Stopping redis     ... done
```

Elimino el escenario:

```console
atlas@olympus:~/docker/guestbook$ docker-compose down
Removing guestbook ... done
Removing redis     ... done
Removing network guestbook_default
```

### Ejemplo 2: Despliegue de la aplicación temperaturas

Creo el directorio y entro:

```console
atlas@olympus:~/docker$ mkdir temperaturas
atlas@olympus:~/docker$ cd temperaturas/
atlas@olympus:~/docker/temperaturas$
```

Creo el siguiente `docker-compose.yml`:

```console
version: '3.1'
services:
  frontend:
    container_name: temperaturas-frontend
    image: iesgn/temperaturas_frontend
    restart: always
    ports:
      - 80:3000
    depends_on:
      - backend
  backend:
    container_name: temperaturas-backend
    image: iesgn/temperaturas_backend
    restart: always
```

Lanzo el escenario:

```console
atlas@olympus:~/docker/temperaturas$ docker-compose up -d
Creating network "temperaturas_default" with the default driver
Creating temperaturas-backend ... done
Creating temperaturas-frontend ... done
```

Listo los contenedores:

```console
atlas@olympus:~/docker/temperaturas$ docker-compose ps
        Name               Command       State          Ports
---------------------------------------------------------------------
temperaturas-backend    python3 app.py   Up      5000/tcp
temperaturas-frontend   python3 app.py   Up      0.0.0.0:80->3000/tcp
```

Pruebo que funciona:

![temperaturascompose](https://i.imgur.com/9j2Vfft.png)

Elimino el escenario:

```console
atlas@olympus:~/docker/temperaturas$ docker-compose down
Stopping temperaturas-frontend ... done
Stopping temperaturas-backend  ... done
Removing temperaturas-frontend ... done
Removing temperaturas-backend  ... done
Removing network temperaturas_default
```

### Ejemplo 3: Despliegue de WordPress + MariaDB

#### WP con volúmenes

Creo el directorio y entro:

```console
atlas@olympus:~/docker$ mkdir wp_vol
atlas@olympus:~/docker$ cd wp_vol
atlas@olympus:~/docker/wp_vol$
```

Creo el siguiente `docker-compose.yml`:

```console
version: '3.1'
services:
  wordpress:
    container_name: servidor_wp
    image: wordpress
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: user_wp
      WORDPRESS_DB_PASSWORD: asdasd
      WORDPRESS_DB_NAME: bd_wp
    ports:
      - 80:80
    volumes:
      - wordpress_data:/var/www/html/wp-content
  db:
    container_name: servidor_mysql
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: bd_wp
      MYSQL_USER: user_wp
      MYSQL_PASSWORD: asdasd
      MYSQL_ROOT_PASSWORD: asdasd
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    wordpress_data:
    mariadb_data:
```

Lanzo el escenario:

```console
atlas@olympus:~/docker/wp_vol$ docker-compose up -d
Creating network "wp_vol_default" with the default driver
Creating volume "wp_vol_wordpress_data" with default driver
Creating volume "wp_vol_mariadb_data" with default driver
Creating servidor_wp    ... done
Creating servidor_mysql ... done
```

Listo los contenedores:

```console
atlas@olympus:~/docker/wp_vol$ docker-compose ps
     Name                   Command               State         Ports
----------------------------------------------------------------------------
servidor_mysql   docker-entrypoint.sh mariadbd    Up      3306/tcp
servidor_wp      docker-entrypoint.sh apach ...   Up      0.0.0.0:80->80/tcp
```

Pruebo que funciona:

![wpvolcompose](https://i.imgur.com/ZCRrRW2.png)

Paro los contenedores:

```console
atlas@olympus:~/docker/wp_vol$ docker-compose stop
Stopping servidor_mysql ... done
Stopping servidor_wp    ... done
```

Borro los contenedores:

```console
atlas@olympus:~/docker/wp_vol$ docker-compose rm
Going to remove servidor_mysql, servidor_wp
Are you sure? [yN] y
Removing servidor_mysql ... done
Removing servidor_wp    ... done
```

Elimino el escenario:

```console
atlas@olympus:~/docker/wp_vol$ docker-compose down -v
Removing network wp_vol_default
Removing volume wp_vol_wordpress_data
Removing volume wp_vol_mariadb_data
```

#### WP con bind mount

Creo el directorio y entro:

```console
atlas@olympus:~/docker$ mkdir wp_bind_mount
atlas@olympus:~/docker$ cd wp_bind_mount
atlas@olympus:~/docker/wp_bind_mount$
```

Creo el siguiente `docker-compose.yml`:

```console
version: '3.1'
services:
  wordpress:
    container_name: servidor_wp
    image: wordpress
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: user_wp
      WORDPRESS_DB_PASSWORD: asdasd
      WORDPRESS_DB_NAME: bd_wp
    ports:
      - 80:80
    volumes:
      - ./wordpress:/var/www/html/wp-content
  db:
    container_name: servidor_mysql
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: bd_wp
      MYSQL_USER: user_wp
      MYSQL_PASSWORD: asdasd
      MYSQL_ROOT_PASSWORD: asdasd
    volumes:
      - ./mysql:/var/lib/mysql
```

Lanzo el escenario:

```console
atlas@olympus:~/docker/wp_bind_mount$ docker-compose up -d
Creating network "wp_bind_mount_default" with the default driver
Creating servidor_wp    ... done
Creating servidor_mysql ... done
```

Listo los contenedores:

```console
atlas@olympus:~/docker/wp_bind_mount$ docker-compose ps
     Name                   Command               State         Ports
----------------------------------------------------------------------------
servidor_mysql   docker-entrypoint.sh mariadbd    Up      3306/tcp
servidor_wp      docker-entrypoint.sh apach ...   Up      0.0.0.0:80->80/tcp
```

Pruebo que funciona:

![wpbindmountcompose](https://i.imgur.com/z5HoUdk.png)

Elimino el escenario:

```console
atlas@olympus:~/docker/wp_bind_mount$ docker-compose down -v
Stopping servidor_mysql ... done
Stopping servidor_wp    ... done
Removing servidor_mysql ... done
Removing servidor_wp    ... done
Removing network wp_bind_mount_default
```

### Ejemplo 4: Despliegue de tomcat + nginx

Creo el directorio y entro:

```console
atlas@olympus:~/docker$ mkdir tomcat
atlas@olympus:~/docker$ cd tomcat
atlas@olympus:~/docker/tomcat$
```

Creo el siguiente `docker-compose.yml`:

```console
version: '3.1'
services:
  aplicacionjava:
    container_name: tomcat
    image: tomcat:9.0
    restart: always
    volumes:
      - ./sample.war:/usr/local/tomcat/webapps/sample.war:ro
  proxy:
    container_name: nginx
    image: nginx
    ports:
      - 80:80
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf:ro
```

Obtengo los 2 ficheros que necesito:

```console
atlas@olympus:~/docker/tomcat$ wget https://raw.githubusercontent.com/iesgn/curso_docker_2021/main/ejemplos/sesion5/ejemplo4/default.conf
--2022-06-16 09:27:24--  https://raw.githubusercontent.com/iesgn/curso_docker_2021/main/ejemplos/sesion5/ejemplo4/default.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.111.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 387 [text/plain]
Saving to: ‘default.conf’

default.conf                                100%[=========================================================================================>]     387  --.-KB/s    in 0s

2022-06-16 09:27:24 (16.0 MB/s) - ‘default.conf’ saved [387/387]

atlas@olympus:~/docker/tomcat$ wget https://github.com/iesgn/curso_docker_2021/raw/main/ejemplos/sesion5/ejemplo4/sample.war
--2022-06-16 09:28:18--  https://github.com/iesgn/curso_docker_2021/raw/main/ejemplos/sesion5/ejemplo4/sample.war
Resolving github.com (github.com)... 140.82.121.4
Connecting to github.com (github.com)|140.82.121.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/iesgn/curso_docker_2021/main/ejemplos/sesion5/ejemplo4/sample.war [following]
--2022-06-16 09:28:18--  https://raw.githubusercontent.com/iesgn/curso_docker_2021/main/ejemplos/sesion5/ejemplo4/sample.war
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.110.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4606 (4.5K) [application/octet-stream]
Saving to: ‘sample.war’

sample.war                                  100%[=========================================================================================>]   4.50K  --.-KB/s    in 0s

2022-06-16 09:28:19 (30.0 MB/s) - ‘sample.war’ saved [4606/4606]
```

Lanzo el escenario:

```console
atlas@olympus:~/docker/tomcat$ docker-compose up -d
Creating network "tomcat_default" with the default driver
Creating tomcat ... done
Creating nginx  ... done
```

Compruebo que los contenedores están activos:

```console
atlas@olympus:~/docker/tomcat$ docker-compose ps
 Name               Command               State         Ports
--------------------------------------------------------------------
nginx    /docker-entrypoint.sh ngin ...   Up      0.0.0.0:80->80/tcp
tomcat   catalina.sh run                  Up      8080/tcp
```

Pruebo que funciona:

![tomcatcompose](https://i.imgur.com/4ah43NP.png)

Elimino el escenario:

```console
atlas@olympus:~/docker/tomcat$ docker-compose down -v
Stopping nginx  ... done
Stopping tomcat ... done
Removing nginx  ... done
Removing tomcat ... done
Removing network tomcat_default
```

### Ejemplos reales de despliegues usando docker-compose

* Jitsi
* Bitnami
* Guacamole

### Ejercicios multicontenedor

Creo el directorio y entro:

```console
atlas@olympus:~/docker$ mkdir prestashop
atlas@olympus:~/docker$ cd prestashop
atlas@olympus:~/docker/prestashop$
```

Descargo el `docker-compose.yml` oficial de bitnami:

```console
curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-prestashop/master/docker-compose.yml > docker-compose.yml
```

Lo muestro sin modificar:

```console
atlas@olympus:~/docker/prestashop$ cat docker-compose.yml
version: '2'
services:
  mariadb:
    image: docker.io/bitnami/mariadb:10.6
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_prestashop
      - MARIADB_DATABASE=bitnami_prestashop
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
  prestashop:
    image: docker.io/bitnami/prestashop:1.7
    ports:
      - '80:8080'
      - '443:8443'
    environment:
      - PRESTASHOP_HOST=localhost
      - PRESTASHOP_DATABASE_HOST=mariadb
      - PRESTASHOP_DATABASE_PORT_NUMBER=3306
      - PRESTASHOP_DATABASE_USER=bn_prestashop
      - PRESTASHOP_DATABASE_NAME=bitnami_prestashop
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - 'prestashop_data:/bitnami/prestashop'
    depends_on:
      - mariadb
volumes:
  mariadb_data:
    driver: local
  prestashop_data:
    driver: local
```

Lo muestro modificado:

![prestashopmodificadocompose](https://i.imgur.com/9PSPhPy.png)

Lanzo el escenario:

```console
atlas@olympus:~/docker/prestashop$ docker-compose up -d
Creating network "prestashop_default" with the default driver
Creating volume "prestashop_mariadb_data" with local driver
Creating volume "prestashop_prestashop_data" with local driver
Creating prestashop_mariadb_1 ... done
Creating prestashop_prestashop_1 ... done
```

Muestro los contenedores funcionando:

![prestashopcontenedores](https://i.imgur.com/0oVJrWK.png)

Muestro la web funcionando:

![prestashopfuncionando](https://i.imgur.com/Tv1Fwim.png)

## Creación de imágenes

### Creación de imágenes a partir de un contenedor

Lanzo un contenedor:

```console
atlas@olympus:~/docker$ docker run -it --name contenedor debian bash
root@d42a140c8da0:/#
```

Hago modificaciones:

```console
root@d42a140c8da0:/# apt update
Get:1 http://security.debian.org/debian-security bullseye-security InRelease [44.1 kB]
Get:2 http://deb.debian.org/debian bullseye InRelease [116 kB]
Get:3 http://deb.debian.org/debian bullseye-updates InRelease [39.4 kB]
Get:4 http://security.debian.org/debian-security bullseye-security/main amd64 Packages [154 kB]
Get:5 http://deb.debian.org/debian bullseye/main amd64 Packages [8182 kB]
Get:6 http://deb.debian.org/debian bullseye-updates/main amd64 Packages [2592 B]
Fetched 8539 kB in 6s (1401 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
1 package can be upgraded. Run 'apt list --upgradable' to see it.
root@d42a140c8da0:/# apt install apache2 -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils bzip2 ca-certificates file libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libbrotli1 libcurl4 libexpat1
  libgdbm-compat4 libgdbm6 libgpm2 libicu67 libjansson4 libldap-2.4-2 libldap-common liblua5.3-0 libmagic-mgc libmagic1 libncurses6 libncursesw6 libnghttp2-14 libperl5.32
  libprocps8 libpsl5 librtmp1 libsasl2-2 libsasl2-modules libsasl2-modules-db libsqlite3-0 libssh2-1 libxml2 mailcap media-types mime-support netbase openssl perl
  perl-modules-5.32 procps psmisc publicsuffix ssl-cert xz-utils
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser bzip2-doc gdbm-l10n gpm sensible-utils libsasl2-modules-gssapi-mit
  | libsasl2-modules-gssapi-heimdal libsasl2-modules-ldap libsasl2-modules-otp libsasl2-modules-sql perl-doc libterm-readline-gnu-perl | libterm-readline-perl-perl make
  libtap-harness-archive-perl
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils bzip2 ca-certificates file libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libbrotli1 libcurl4 libexpat1
  libgdbm-compat4 libgdbm6 libgpm2 libicu67 libjansson4 libldap-2.4-2 libldap-common liblua5.3-0 libmagic-mgc libmagic1 libncurses6 libncursesw6 libnghttp2-14 libperl5.32
  libprocps8 libpsl5 librtmp1 libsasl2-2 libsasl2-modules libsasl2-modules-db libsqlite3-0 libssh2-1 libxml2 mailcap media-types mime-support netbase openssl perl
  perl-modules-5.32 procps psmisc publicsuffix ssl-cert xz-utils
0 upgraded, 49 newly installed, 0 to remove and 1 not upgraded.
Need to get 24.6 MB of archives.
After this operation, 111 MB of additional disk space will be used.
Get:1 http://security.debian.org/debian-security bullseye-security/main amd64 libldap-2.4-2 amd64 2.4.57+dfsg-3+deb11u1 [232 kB]
Get:2 http://security.debian.org/debian-security bullseye-security/main amd64 libxml2 amd64 2.9.10+dfsg-6.7+deb11u2 [692 kB]
Get:3 http://deb.debian.org/debian bullseye/main amd64 perl-modules-5.32 all 5.32.1-4+deb11u2 [2823 kB]
Get:4 http://security.debian.org/debian-security bullseye-security/main amd64 openssl amd64 1.1.1n-0+deb11u2 [852 kB]
Get:5 http://security.debian.org/debian-security bullseye-security/main amd64 xz-utils amd64 5.2.5-2.1~deb11u1 [220 kB]
Get:6 http://security.debian.org/debian-security bullseye-security/main amd64 libldap-common all 2.4.57+dfsg-3+deb11u1 [95.8 kB]
Get:7 http://deb.debian.org/debian bullseye/main amd64 libgdbm6 amd64 1.19-2 [64.9 kB]
Setting up libgdbm-compat4:amd64 (1.19-2) ...
Setting up libperl5.32:amd64 (5.32.1-4+deb11u2) ...
Setting up procps (2:3.3.17-5) ...
Setting up libcurl4:amd64 (7.74.0-1.3+deb11u1) ...
Setting up apache2-utils (2.4.53-1~deb11u1) ...
Setting up perl (5.32.1-4+deb11u2) ...
Setting up mailcap (3.69) ...
Setting up mime-support (3.66) ...
Setting up apache2-bin (2.4.53-1~deb11u1) ...
Setting up apache2 (2.4.53-1~deb11u1) ...
Enabling module mpm_event.
Enabling module authz_core.
Enabling module authz_host.
Enabling module authn_core.
Enabling module auth_basic.
Enabling module access_compat.
Enabling module authn_file.
Enabling module authz_user.
Enabling module alias.
Enabling module dir.
Enabling module autoindex.
Enabling module env.
Enabling module mime.
Enabling module negotiation.
Enabling module setenvif.
Enabling module filter.
Enabling module deflate.
Enabling module status.
Enabling module reqtimeout.
Enabling conf charset.
Enabling conf localized-error-pages.
Enabling conf other-vhosts-access-log.
Enabling conf security.
Enabling conf serve-cgi-bin.
Enabling site 000-default.
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Processing triggers for libc-bin (2.31-13+deb11u3) ...
Processing triggers for ca-certificates (20210119) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
root@d42a140c8da0:/# echo "<h1>Curso Docker</h1>" > /var/www/html/index.html
root@d42a140c8da0:/# exit
exit
```

Creo una nueva imagen partiendo de los cambios en el contenedor. Esta imagen será el resultado de las capas base + cambios:

```console
atlas@olympus:~$ docker commit contenedor adrianjaramillo/myapache2:v1
sha256:4d7daf41fb1034a6667c2464ac9bcf809d3c6460af908120f755672d66bc6433
```

Muestro que ya la tengo localmente:

![debianmodificado](https://i.imgur.com/ObCrzY4.png)

Al lanzar contenedores desde la imagen modificada **estamos obligados a indicar el proceso a iniciar**, ya que por defecto se ejecuta el de la imagen base y no podemos cambiarlo *(por ahora)*:

```console
atlas@olympus:~$ docker run -d -p 8080:80 \
             --name servidor_web \
             adrianjaramillo/myapache2:v1 \
             bash -c "apache2ctl -D FOREGROUND"
fed43969a4a81ddfe4473097201a271fdd10aa7ee0053caf11193c12e540e321
```

Muestro que funciona:

![imagenmodificadacontenedor](https://i.imgur.com/epmOg1K.png)

### Creación de imágenes a partir de un Dockerfile

Creo un directorio donde estará el Dockerfile y lo que se necesite:

```console
atlas@olympus:~/docker$ mkdir build1
atlas@olympus:~/docker$ cd build1
atlas@olympus:~/docker/build1$
```

El Dockerfile será:

```console
FROM debian:buster-slim
MAINTAINER Adrián Jaramillo Rodríguez "adristudy@gmail.com"
RUN apt-get update && apt-get install -y apache2
COPY index.html /var/www/html/
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Me bajo un `index.html` de prueba:

```console
atlas@olympus:~/docker/build1$ wget https://gist.githubusercontent.com/chrisvfritz/bc010e6ed25b802da7eb/raw/18eaa48addae7e3021f6bcea03b7a6557e3f0132/index.html
--2022-06-16 18:53:29--  https://gist.githubusercontent.com/chrisvfritz/bc010e6ed25b802da7eb/raw/18eaa48addae7e3021f6bcea03b7a6557e3f0132/index.html
Resolving gist.githubusercontent.com (gist.githubusercontent.com)... 185.199.111.133, 185.199.110.133, 185.199.108.133, ...
Connecting to gist.githubusercontent.com (gist.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 285 [text/plain]
Saving to: ‘index.html’

index.html                                  100%[=========================================================================================>]     285  --.-KB/s    in 0s

2022-06-16 18:53:30 (11.6 MB/s) - ‘index.html’ saved [285/285]
```

Al final, acabaremos con la siguiente estructura:

```console
atlas@olympus:~/docker/build1$ ls -la
total 16
drwxr-xr-x  2 atlas atlas 4096 Jun 16 18:53 .
drwxr-xr-x 13 atlas atlas 4096 Jun 16 18:37 ..
-rw-r--r--  1 atlas atlas  216 Jun 16 18:50 Dockerfile
-rw-r--r--  1 atlas atlas  285 Jun 16 18:53 index.html
```

Genero la imagen:

```console
atlas@olympus:~/docker/build1$ docker build -t adrianjaramillo/myapache2:v2 .
Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM debian:buster-slim
 ---> c2303a498941
Step 2/5 : MAINTAINER Adrián Jaramillo Rodríguez "adristudy@gmail.com"
 ---> Using cache
 ---> 31ec41690e42
Step 3/5 : RUN apt-get update && apt-get install -y apache2
 ---> Running in 82497c561473
Get:1 http://deb.debian.org/debian buster InRelease [122 kB]
Get:2 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
Get:3 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
Get:4 http://security.debian.org/debian-security buster/updates/main amd64 Packages [328 kB]
Get:5 http://deb.debian.org/debian buster/main amd64 Packages [7911 kB]
Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [8788 B]
Fetched 8487 kB in 1min 9s (123 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils bzip2 ca-certificates file
  krb5-locales libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap
  libbrotli1 libcurl4 libexpat1 libgdbm-compat4 libgdbm6 libgpm2
  libgssapi-krb5-2 libicu63 libjansson4 libk5crypto3 libkeyutils1 libkrb5-3
  libkrb5support0 libldap-2.4-2 libldap-common liblua5.2-0 libmagic-mgc
  libmagic1 libncurses6 libnghttp2-14 libperl5.28 libprocps7 libpsl5 librtmp1
  libsasl2-2 libsasl2-modules libsasl2-modules-db libsqlite3-0 libssh2-1
  libssl1.1 libxml2 lsb-base mime-support netbase openssl perl
  perl-modules-5.28 procps psmisc publicsuffix ssl-cert xz-utils
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser
  bzip2-doc gdbm-l10n gpm krb5-doc krb5-user sensible-utils
  libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal
  libsasl2-modules-ldap libsasl2-modules-otp libsasl2-modules-sql perl-doc
  libterm-readline-gnu-perl | libterm-readline-perl-perl make libb-debug-perl
  liblocale-codes-perl openssl-blacklist
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils bzip2 ca-certificates file
  krb5-locales libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap
  libbrotli1 libcurl4 libexpat1 libgdbm-compat4 libgdbm6 libgpm2
  libgssapi-krb5-2 libicu63 libjansson4 libk5crypto3 libkeyutils1 libkrb5-3
  libkrb5support0 libldap-2.4-2 libldap-common liblua5.2-0 libmagic-mgc
  libmagic1 libncurses6 libnghttp2-14 libperl5.28 libprocps7 libpsl5 librtmp1
  libsasl2-2 libsasl2-modules libsasl2-modules-db libsqlite3-0 libssh2-1
  libssl1.1 libxml2 lsb-base mime-support netbase openssl perl
  perl-modules-5.28 procps psmisc publicsuffix ssl-cert xz-utils
0 upgraded, 54 newly installed, 0 to remove and 1 not upgraded.
Need to get 25.6 MB of archives.
After this operation, 112 MB of additional disk space will be used.
Get:1 http://security.debian.org/debian-security buster/updates/main amd64 libssl1.1 amd64 1.1.1n-0+deb10u2 [1551 kB]
Get:2 http://deb.debian.org/debian buster/main amd64 perl-modules-5.28 all 5.28.1-6+deb10u1 [2873 kB]
Get:3 http://deb.debian.org/debian buster/main amd64 libgdbm6 amd64 1.18.1-4 [64.7 kB]
Get:4 http://deb.debian.org/debian buster/main amd64 libgdbm-compat4 amd64 1.18.1-4 [44.1 kB]
Get:5 http://deb.debian.org/debian buster/main amd64 libperl5.28 amd64 5.28.1-6+deb10u1 [3894 kB]
Get:6 http://security.debian.org/debian-security buster/updates/main amd64 libldap-common all 2.4.47+dfsg-3+deb10u7 [90.1 kB]
Get:7 http://security.debian.org/debian-security buster/updates/main amd64 libldap-2.4-2 amd64 2.4.47+dfsg-3+deb10u7 [224 kB]
Get:8 http://security.debian.org/debian-security buster/updates/main amd64 libxml2 amd64 2.9.4+dfsg1-7+deb10u4 [689 kB]
Get:9 http://security.debian.org/debian-security buster/updates/main amd64 xz-utils amd64 5.2.4-1+deb10u1 [183 kB]
Get:10 http://security.debian.org/debian-security buster/updates/main amd64 openssl amd64 1.1.1n-0+deb10u2 [855 kB]
Get:11 http://deb.debian.org/debian buster/main amd64 perl amd64 5.28.1-6+deb10u1 [204 kB]
Get:12 http://deb.debian.org/debian buster/main amd64 libapr1 amd64 1.6.5-1+b1 [102 kB]
Get:13 http://deb.debian.org/debian buster/main amd64 libexpat1 amd64 2.2.6-2+deb10u4 [108 kB]
Get:14 http://deb.debian.org/debian buster/main amd64 libaprutil1 amd64 1.6.1-4 [91.8 kB]
Get:15 http://deb.debian.org/debian buster/main amd64 libsqlite3-0 amd64 3.27.2-3+deb10u1 [641 kB]
Get:16 http://deb.debian.org/debian buster/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.1-4 [18.7 kB]
Get:17 http://deb.debian.org/debian buster/main amd64 libsasl2-modules-db amd64 2.1.27+dfsg-1+deb10u2 [69.2 kB]
Get:18 http://deb.debian.org/debian buster/main amd64 libsasl2-2 amd64 2.1.27+dfsg-1+deb10u2 [106 kB]
Get:19 http://deb.debian.org/debian buster/main amd64 libaprutil1-ldap amd64 1.6.1-4 [16.8 kB]
Get:20 http://deb.debian.org/debian buster/main amd64 libbrotli1 amd64 1.0.7-2+deb10u1 [269 kB]
Get:21 http://deb.debian.org/debian buster/main amd64 libkeyutils1 amd64 1.6-6 [15.0 kB]
Get:22 http://deb.debian.org/debian buster/main amd64 libkrb5support0 amd64 1.17-3+deb10u3 [65.8 kB]
Get:23 http://deb.debian.org/debian buster/main amd64 libk5crypto3 amd64 1.17-3+deb10u3 [122 kB]
Get:24 http://deb.debian.org/debian buster/main amd64 libkrb5-3 amd64 1.17-3+deb10u3 [370 kB]
Get:25 http://deb.debian.org/debian buster/main amd64 libgssapi-krb5-2 amd64 1.17-3+deb10u3 [158 kB]
Get:26 http://deb.debian.org/debian buster/main amd64 libnghttp2-14 amd64 1.36.0-2+deb10u1 [85.0 kB]
Get:27 http://deb.debian.org/debian buster/main amd64 libpsl5 amd64 0.20.2-2 [53.7 kB]
Get:28 http://deb.debian.org/debian buster/main amd64 librtmp1 amd64 2.4+20151223.gitfa8646d.1-2 [60.5 kB]
Get:29 http://deb.debian.org/debian buster/main amd64 libssh2-1 amd64 1.8.0-2.1 [140 kB]
Get:30 http://deb.debian.org/debian buster/main amd64 libcurl4 amd64 7.64.0-4+deb10u2 [332 kB]
Get:31 http://deb.debian.org/debian buster/main amd64 libjansson4 amd64 2.12-1 [38.0 kB]
Get:32 http://deb.debian.org/debian buster/main amd64 liblua5.2-0 amd64 5.2.4-1.1+b2 [110 kB]
Get:33 http://deb.debian.org/debian buster/main amd64 libicu63 amd64 63.1-6+deb10u3 [8293 kB]
Get:34 http://deb.debian.org/debian buster/main amd64 apache2-bin amd64 2.4.38-3+deb10u7 [1308 kB]
Get:35 http://deb.debian.org/debian buster/main amd64 apache2-data all 2.4.38-3+deb10u7 [165 kB]
Get:36 http://deb.debian.org/debian buster/main amd64 apache2-utils amd64 2.4.38-3+deb10u7 [237 kB]
Get:37 http://deb.debian.org/debian buster/main amd64 lsb-base all 10.2019051400 [28.4 kB]
Get:38 http://deb.debian.org/debian buster/main amd64 mime-support all 3.62 [37.2 kB]
Get:39 http://deb.debian.org/debian buster/main amd64 libncurses6 amd64 6.1+20181013-2+deb10u2 [102 kB]
Get:40 http://deb.debian.org/debian buster/main amd64 libprocps7 amd64 2:3.3.15-2 [61.7 kB]
Get:41 http://deb.debian.org/debian buster/main amd64 procps amd64 2:3.3.15-2 [259 kB]
Get:42 http://deb.debian.org/debian buster/main amd64 apache2 amd64 2.4.38-3+deb10u7 [252 kB]
Get:43 http://deb.debian.org/debian buster/main amd64 netbase all 5.6 [19.4 kB]
Get:44 http://deb.debian.org/debian buster/main amd64 bzip2 amd64 1.0.6-9.2~deb10u1 [48.4 kB]
Get:45 http://deb.debian.org/debian buster/main amd64 libmagic-mgc amd64 1:5.35-4+deb10u2 [242 kB]
Get:46 http://deb.debian.org/debian buster/main amd64 libmagic1 amd64 1:5.35-4+deb10u2 [118 kB]
Get:47 http://deb.debian.org/debian buster/main amd64 file amd64 1:5.35-4+deb10u2 [66.4 kB]
Get:48 http://deb.debian.org/debian buster/main amd64 krb5-locales all 1.17-3+deb10u3 [95.5 kB]
Get:49 http://deb.debian.org/debian buster/main amd64 ca-certificates all 20200601~deb10u2 [166 kB]
Get:50 http://deb.debian.org/debian buster/main amd64 libgpm2 amd64 1.20.7-5 [35.1 kB]
Get:51 http://deb.debian.org/debian buster/main amd64 libsasl2-modules amd64 2.1.27+dfsg-1+deb10u2 [104 kB]
Get:52 http://deb.debian.org/debian buster/main amd64 psmisc amd64 23.2-1+deb10u1 [126 kB]
Get:53 http://deb.debian.org/debian buster/main amd64 publicsuffix all 20211109.1735-0+deb10u1 [125 kB]
Get:54 http://deb.debian.org/debian buster/main amd64 ssl-cert all 1.0.39 [20.8 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 25.6 MB in 1min 54s (224 kB/s)
Selecting previously unselected package perl-modules-5.28.
(Reading database ... 6460 files and directories currently installed.)
Preparing to unpack .../00-perl-modules-5.28_5.28.1-6+deb10u1_all.deb ...
Unpacking perl-modules-5.28 (5.28.1-6+deb10u1) ...
Selecting previously unselected package libgdbm6:amd64.
Preparing to unpack .../01-libgdbm6_1.18.1-4_amd64.deb ...
Unpacking libgdbm6:amd64 (1.18.1-4) ...
Selecting previously unselected package libgdbm-compat4:amd64.
Preparing to unpack .../02-libgdbm-compat4_1.18.1-4_amd64.deb ...
Unpacking libgdbm-compat4:amd64 (1.18.1-4) ...
Selecting previously unselected package libperl5.28:amd64.
Preparing to unpack .../03-libperl5.28_5.28.1-6+deb10u1_amd64.deb ...
Unpacking libperl5.28:amd64 (5.28.1-6+deb10u1) ...
Selecting previously unselected package perl.
Preparing to unpack .../04-perl_5.28.1-6+deb10u1_amd64.deb ...
Unpacking perl (5.28.1-6+deb10u1) ...
Selecting previously unselected package libapr1:amd64.
Preparing to unpack .../05-libapr1_1.6.5-1+b1_amd64.deb ...
Unpacking libapr1:amd64 (1.6.5-1+b1) ...
Selecting previously unselected package libexpat1:amd64.
Preparing to unpack .../06-libexpat1_2.2.6-2+deb10u4_amd64.deb ...
Unpacking libexpat1:amd64 (2.2.6-2+deb10u4) ...
Selecting previously unselected package libssl1.1:amd64.
Preparing to unpack .../07-libssl1.1_1.1.1n-0+deb10u2_amd64.deb ...
Unpacking libssl1.1:amd64 (1.1.1n-0+deb10u2) ...
Selecting previously unselected package libaprutil1:amd64.
Preparing to unpack .../08-libaprutil1_1.6.1-4_amd64.deb ...
Unpacking libaprutil1:amd64 (1.6.1-4) ...
Selecting previously unselected package libsqlite3-0:amd64.
Preparing to unpack .../09-libsqlite3-0_3.27.2-3+deb10u1_amd64.deb ...
Unpacking libsqlite3-0:amd64 (3.27.2-3+deb10u1) ...
Selecting previously unselected package libaprutil1-dbd-sqlite3:amd64.
Preparing to unpack .../10-libaprutil1-dbd-sqlite3_1.6.1-4_amd64.deb ...
Unpacking libaprutil1-dbd-sqlite3:amd64 (1.6.1-4) ...
Selecting previously unselected package libsasl2-modules-db:amd64.
Preparing to unpack .../11-libsasl2-modules-db_2.1.27+dfsg-1+deb10u2_amd64.deb ...
Unpacking libsasl2-modules-db:amd64 (2.1.27+dfsg-1+deb10u2) ...
Selecting previously unselected package libsasl2-2:amd64.
Preparing to unpack .../12-libsasl2-2_2.1.27+dfsg-1+deb10u2_amd64.deb ...
Unpacking libsasl2-2:amd64 (2.1.27+dfsg-1+deb10u2) ...
Selecting previously unselected package libldap-common.
Preparing to unpack .../13-libldap-common_2.4.47+dfsg-3+deb10u7_all.deb ...
Unpacking libldap-common (2.4.47+dfsg-3+deb10u7) ...
Selecting previously unselected package libldap-2.4-2:amd64.
Preparing to unpack .../14-libldap-2.4-2_2.4.47+dfsg-3+deb10u7_amd64.deb ...
Unpacking libldap-2.4-2:amd64 (2.4.47+dfsg-3+deb10u7) ...
Selecting previously unselected package libaprutil1-ldap:amd64.
Preparing to unpack .../15-libaprutil1-ldap_1.6.1-4_amd64.deb ...
Unpacking libaprutil1-ldap:amd64 (1.6.1-4) ...
Selecting previously unselected package libbrotli1:amd64.
Preparing to unpack .../16-libbrotli1_1.0.7-2+deb10u1_amd64.deb ...
Unpacking libbrotli1:amd64 (1.0.7-2+deb10u1) ...
Selecting previously unselected package libkeyutils1:amd64.
Preparing to unpack .../17-libkeyutils1_1.6-6_amd64.deb ...
Unpacking libkeyutils1:amd64 (1.6-6) ...
Selecting previously unselected package libkrb5support0:amd64.
Preparing to unpack .../18-libkrb5support0_1.17-3+deb10u3_amd64.deb ...
Unpacking libkrb5support0:amd64 (1.17-3+deb10u3) ...
Selecting previously unselected package libk5crypto3:amd64.
Preparing to unpack .../19-libk5crypto3_1.17-3+deb10u3_amd64.deb ...
Unpacking libk5crypto3:amd64 (1.17-3+deb10u3) ...
Selecting previously unselected package libkrb5-3:amd64.
Preparing to unpack .../20-libkrb5-3_1.17-3+deb10u3_amd64.deb ...
Unpacking libkrb5-3:amd64 (1.17-3+deb10u3) ...
Selecting previously unselected package libgssapi-krb5-2:amd64.
Preparing to unpack .../21-libgssapi-krb5-2_1.17-3+deb10u3_amd64.deb ...
Unpacking libgssapi-krb5-2:amd64 (1.17-3+deb10u3) ...
Selecting previously unselected package libnghttp2-14:amd64.
Preparing to unpack .../22-libnghttp2-14_1.36.0-2+deb10u1_amd64.deb ...
Unpacking libnghttp2-14:amd64 (1.36.0-2+deb10u1) ...
Selecting previously unselected package libpsl5:amd64.
Preparing to unpack .../23-libpsl5_0.20.2-2_amd64.deb ...
Unpacking libpsl5:amd64 (0.20.2-2) ...
Selecting previously unselected package librtmp1:amd64.
Preparing to unpack .../24-librtmp1_2.4+20151223.gitfa8646d.1-2_amd64.deb ...
Unpacking librtmp1:amd64 (2.4+20151223.gitfa8646d.1-2) ...
Selecting previously unselected package libssh2-1:amd64.
Preparing to unpack .../25-libssh2-1_1.8.0-2.1_amd64.deb ...
Unpacking libssh2-1:amd64 (1.8.0-2.1) ...
Selecting previously unselected package libcurl4:amd64.
Preparing to unpack .../26-libcurl4_7.64.0-4+deb10u2_amd64.deb ...
Unpacking libcurl4:amd64 (7.64.0-4+deb10u2) ...
Selecting previously unselected package libjansson4:amd64.
Preparing to unpack .../27-libjansson4_2.12-1_amd64.deb ...
Unpacking libjansson4:amd64 (2.12-1) ...
Selecting previously unselected package liblua5.2-0:amd64.
Preparing to unpack .../28-liblua5.2-0_5.2.4-1.1+b2_amd64.deb ...
Unpacking liblua5.2-0:amd64 (5.2.4-1.1+b2) ...
Selecting previously unselected package libicu63:amd64.
Preparing to unpack .../29-libicu63_63.1-6+deb10u3_amd64.deb ...
Unpacking libicu63:amd64 (63.1-6+deb10u3) ...
Selecting previously unselected package libxml2:amd64.
Preparing to unpack .../30-libxml2_2.9.4+dfsg1-7+deb10u4_amd64.deb ...
Unpacking libxml2:amd64 (2.9.4+dfsg1-7+deb10u4) ...
Selecting previously unselected package apache2-bin.
Preparing to unpack .../31-apache2-bin_2.4.38-3+deb10u7_amd64.deb ...
Unpacking apache2-bin (2.4.38-3+deb10u7) ...
Selecting previously unselected package apache2-data.
Preparing to unpack .../32-apache2-data_2.4.38-3+deb10u7_all.deb ...
Unpacking apache2-data (2.4.38-3+deb10u7) ...
Selecting previously unselected package apache2-utils.
Preparing to unpack .../33-apache2-utils_2.4.38-3+deb10u7_amd64.deb ...
Unpacking apache2-utils (2.4.38-3+deb10u7) ...
Selecting previously unselected package lsb-base.
Preparing to unpack .../34-lsb-base_10.2019051400_all.deb ...
Unpacking lsb-base (10.2019051400) ...
Selecting previously unselected package mime-support.
Preparing to unpack .../35-mime-support_3.62_all.deb ...
Unpacking mime-support (3.62) ...
Selecting previously unselected package libncurses6:amd64.
Preparing to unpack .../36-libncurses6_6.1+20181013-2+deb10u2_amd64.deb ...
Unpacking libncurses6:amd64 (6.1+20181013-2+deb10u2) ...
Selecting previously unselected package libprocps7:amd64.
Preparing to unpack .../37-libprocps7_2%3a3.3.15-2_amd64.deb ...
Unpacking libprocps7:amd64 (2:3.3.15-2) ...
Selecting previously unselected package procps.
Preparing to unpack .../38-procps_2%3a3.3.15-2_amd64.deb ...
Unpacking procps (2:3.3.15-2) ...
Selecting previously unselected package apache2.
Preparing to unpack .../39-apache2_2.4.38-3+deb10u7_amd64.deb ...
Unpacking apache2 (2.4.38-3+deb10u7) ...
Selecting previously unselected package netbase.
Preparing to unpack .../40-netbase_5.6_all.deb ...
Unpacking netbase (5.6) ...
Selecting previously unselected package bzip2.
Preparing to unpack .../41-bzip2_1.0.6-9.2~deb10u1_amd64.deb ...
Unpacking bzip2 (1.0.6-9.2~deb10u1) ...
Selecting previously unselected package libmagic-mgc.
Preparing to unpack .../42-libmagic-mgc_1%3a5.35-4+deb10u2_amd64.deb ...
Unpacking libmagic-mgc (1:5.35-4+deb10u2) ...
Selecting previously unselected package libmagic1:amd64.
Preparing to unpack .../43-libmagic1_1%3a5.35-4+deb10u2_amd64.deb ...
Unpacking libmagic1:amd64 (1:5.35-4+deb10u2) ...
Selecting previously unselected package file.
Preparing to unpack .../44-file_1%3a5.35-4+deb10u2_amd64.deb ...
Unpacking file (1:5.35-4+deb10u2) ...
Selecting previously unselected package krb5-locales.
Preparing to unpack .../45-krb5-locales_1.17-3+deb10u3_all.deb ...
Unpacking krb5-locales (1.17-3+deb10u3) ...
Selecting previously unselected package xz-utils.
Preparing to unpack .../46-xz-utils_5.2.4-1+deb10u1_amd64.deb ...
Unpacking xz-utils (5.2.4-1+deb10u1) ...
Selecting previously unselected package openssl.
Preparing to unpack .../47-openssl_1.1.1n-0+deb10u2_amd64.deb ...
Unpacking openssl (1.1.1n-0+deb10u2) ...
Selecting previously unselected package ca-certificates.
Preparing to unpack .../48-ca-certificates_20200601~deb10u2_all.deb ...
Unpacking ca-certificates (20200601~deb10u2) ...
Selecting previously unselected package libgpm2:amd64.
Preparing to unpack .../49-libgpm2_1.20.7-5_amd64.deb ...
Unpacking libgpm2:amd64 (1.20.7-5) ...
Selecting previously unselected package libsasl2-modules:amd64.
Preparing to unpack .../50-libsasl2-modules_2.1.27+dfsg-1+deb10u2_amd64.deb ...
Unpacking libsasl2-modules:amd64 (2.1.27+dfsg-1+deb10u2) ...
Selecting previously unselected package psmisc.
Preparing to unpack .../51-psmisc_23.2-1+deb10u1_amd64.deb ...
Unpacking psmisc (23.2-1+deb10u1) ...
Selecting previously unselected package publicsuffix.
Preparing to unpack .../52-publicsuffix_20211109.1735-0+deb10u1_all.deb ...
Unpacking publicsuffix (20211109.1735-0+deb10u1) ...
Selecting previously unselected package ssl-cert.
Preparing to unpack .../53-ssl-cert_1.0.39_all.deb ...
Unpacking ssl-cert (1.0.39) ...
Setting up perl-modules-5.28 (5.28.1-6+deb10u1) ...
Setting up libexpat1:amd64 (2.2.6-2+deb10u4) ...
Setting up lsb-base (10.2019051400) ...
Setting up libkeyutils1:amd64 (1.6-6) ...
Setting up libpsl5:amd64 (0.20.2-2) ...
Setting up libgpm2:amd64 (1.20.7-5) ...
Setting up mime-support (3.62) ...
Setting up libmagic-mgc (1:5.35-4+deb10u2) ...
Setting up psmisc (23.2-1+deb10u1) ...
Setting up libssl1.1:amd64 (1.1.1n-0+deb10u2) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
Setting up libprocps7:amd64 (2:3.3.15-2) ...
Setting up libbrotli1:amd64 (1.0.7-2+deb10u1) ...
Setting up libsqlite3-0:amd64 (3.27.2-3+deb10u1) ...
Setting up libsasl2-modules:amd64 (2.1.27+dfsg-1+deb10u2) ...
Setting up libnghttp2-14:amd64 (1.36.0-2+deb10u1) ...
Setting up libmagic1:amd64 (1:5.35-4+deb10u2) ...
Setting up libapr1:amd64 (1.6.5-1+b1) ...
Setting up krb5-locales (1.17-3+deb10u3) ...
Setting up file (1:5.35-4+deb10u2) ...
Setting up bzip2 (1.0.6-9.2~deb10u1) ...
Setting up libldap-common (2.4.47+dfsg-3+deb10u7) ...
Setting up libicu63:amd64 (63.1-6+deb10u3) ...
Setting up libjansson4:amd64 (2.12-1) ...
Setting up libkrb5support0:amd64 (1.17-3+deb10u3) ...
Setting up libsasl2-modules-db:amd64 (2.1.27+dfsg-1+deb10u2) ...
Setting up librtmp1:amd64 (2.4+20151223.gitfa8646d.1-2) ...
Setting up libncurses6:amd64 (6.1+20181013-2+deb10u2) ...
Setting up xz-utils (5.2.4-1+deb10u1) ...
update-alternatives: using /usr/bin/xz to provide /usr/bin/lzma (lzma) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/lzma.1.gz because associated file /usr/share/man/man1/xz.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/unlzma.1.gz because associated file /usr/share/man/man1/unxz.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzcat.1.gz because associated file /usr/share/man/man1/xzcat.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzmore.1.gz because associated file /usr/share/man/man1/xzmore.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzless.1.gz because associated file /usr/share/man/man1/xzless.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzdiff.1.gz because associated file /usr/share/man/man1/xzdiff.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzcmp.1.gz because associated file /usr/share/man/man1/xzcmp.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzgrep.1.gz because associated file /usr/share/man/man1/xzgrep.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzegrep.1.gz because associated file /usr/share/man/man1/xzegrep.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzfgrep.1.gz because associated file /usr/share/man/man1/xzfgrep.1.gz (of link group lzma) doesn't exist
Setting up libk5crypto3:amd64 (1.17-3+deb10u3) ...
Setting up libsasl2-2:amd64 (2.1.27+dfsg-1+deb10u2) ...
Setting up liblua5.2-0:amd64 (5.2.4-1.1+b2) ...
Setting up procps (2:3.3.15-2) ...
update-alternatives: using /usr/bin/w.procps to provide /usr/bin/w (w) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/w.1.gz because associated file /usr/share/man/man1/w.procps.1.gz (of link group w) doesn't exist
Setting up libssh2-1:amd64 (1.8.0-2.1) ...
Setting up netbase (5.6) ...
Setting up libkrb5-3:amd64 (1.17-3+deb10u3) ...
Setting up apache2-data (2.4.38-3+deb10u7) ...
Setting up openssl (1.1.1n-0+deb10u2) ...
Setting up publicsuffix (20211109.1735-0+deb10u1) ...
Setting up libxml2:amd64 (2.9.4+dfsg1-7+deb10u4) ...
Setting up libgdbm6:amd64 (1.18.1-4) ...
Setting up libaprutil1:amd64 (1.6.1-4) ...
Setting up libldap-2.4-2:amd64 (2.4.47+dfsg-3+deb10u7) ...
Setting up libaprutil1-ldap:amd64 (1.6.1-4) ...
Setting up libaprutil1-dbd-sqlite3:amd64 (1.6.1-4) ...
Setting up ca-certificates (20200601~deb10u2) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
Updating certificates in /etc/ssl/certs...
137 added, 0 removed; done.
Setting up ssl-cert (1.0.39) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
Setting up libgssapi-krb5-2:amd64 (1.17-3+deb10u3) ...
Setting up libgdbm-compat4:amd64 (1.18.1-4) ...
Setting up libperl5.28:amd64 (5.28.1-6+deb10u1) ...
Setting up libcurl4:amd64 (7.64.0-4+deb10u2) ...
Setting up apache2-utils (2.4.38-3+deb10u7) ...
Setting up perl (5.28.1-6+deb10u1) ...
Setting up apache2-bin (2.4.38-3+deb10u7) ...
Setting up apache2 (2.4.38-3+deb10u7) ...
Enabling module mpm_event.
Enabling module authz_core.
Enabling module authz_host.
Enabling module authn_core.
Enabling module auth_basic.
Enabling module access_compat.
Enabling module authn_file.
Enabling module authz_user.
Enabling module alias.
Enabling module dir.
Enabling module autoindex.
Enabling module env.
Enabling module mime.
Enabling module negotiation.
Enabling module setenvif.
Enabling module filter.
Enabling module deflate.
Enabling module status.
Enabling module reqtimeout.
Enabling conf charset.
Enabling conf localized-error-pages.
Enabling conf other-vhosts-access-log.
Enabling conf security.
Enabling conf serve-cgi-bin.
Enabling site 000-default.
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Processing triggers for libc-bin (2.28-10+deb10u1) ...
Processing triggers for ca-certificates (20200601~deb10u2) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
Removing intermediate container 82497c561473
 ---> 878ba4d423c6
Step 4/5 : COPY index.html /var/www/html/
 ---> 4a7b35abef6e
Step 5/5 : CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
 ---> Running in b4ea0360f338
Removing intermediate container b4ea0360f338
 ---> fa8b78ee9ce4
Successfully built fa8b78ee9ce4
Successfully tagged adrianjaramillo/myapache2:v2
```

Compruebo que tengo la nueva imagen:

![imagengeneradav2](https://i.imgur.com/Gfv0Y8p.png)

Lanzo un contenedor a partir de esta imagen:

```console
atlas@olympus:~/docker/build1$ docker run -d -p 8080:80 --name servidor_web adrianjaramillo/myapache2:v2
83a16f4cd2bfe74313b38c590697c9d940c3a0d2c3e586e84975748c52e79069
```

Pruebo que funciona:

![imagenv2funcionando](https://i.imgur.com/un0Eh9G.png)

### Distribución de imágenes

#### Por fichero

Pasamos la imagen a un `.tar`:

```console
docker save adrianjaramillo/myapache2:v1 > myapache2.tar
```

El fichero generado ya podríamos pasarlo a cualquier sitio.

Para añadir un `.tar` a nuestro repositorio local:

```console
atlas@olympus:~/docker$ docker load -i myapache2.tar
e54f845af4e5: Loading layer [==================================================>]  131.7MB/131.7MB
Loaded image: adrianjaramillo/myapache2:v1
```

#### Por Docker Hub

Me autentico:

```console
atlas@olympus:~/docker$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: adrianjaramillo
Password:
WARNING! Your password will be stored unencrypted in /home/atlas/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

Subimos a Docker Hub:

```console
atlas@olympus:~/docker$ docker push adrianjaramillo/myapache2:v2
The push refers to repository [docker.io/adrianjaramillo/myapache2]
2f99db70a821: Pushed
928f71c072df: Pushed
10e6bc6fdee2: Mounted from library/debian
v2: digest: sha256:f5e64a6aca0b4642827146857d52f1adf6dcfc6932ba8e844c0ded7a003959d7 size: 948
```

Cualquier persona se podría bajar nuestra imagen ahora:

```console
atlas@olympus:~/docker$ docker pull adrianjaramillo/myapache2:v2
v2: Pulling from adrianjaramillo/myapache2
c1ad9731b2c7: Already exists
7b785f89fe92: Pull complete
73b5fa52da1c: Pull complete
Digest: sha256:f5e64a6aca0b4642827146857d52f1adf6dcfc6932ba8e844c0ded7a003959d7
Status: Downloaded newer image for adrianjaramillo/myapache2:v2
docker.io/adrianjaramillo/myapache2:v2
```

### Ejemplo 1: Construcción de imágenes con una página estática

#### Versión 1: Desde una imagen base

Parto de la siguiente estructura:

```console
atlas@olympus:~/docker/ejemplo1build/version1$ ls -la
total 16
drwxr-xr-x 3 atlas atlas 4096 Jun 16 20:44 .
drwxr-xr-x 5 atlas atlas 4096 Jun 16 20:49 ..
-rw-r--r-- 1 atlas atlas  203 Jun 16 20:44 Dockerfile
drwxr-xr-x 4 atlas atlas 4096 Jun 16 20:44 public_html
```

Muestro el `Dockerfile`:

```console
FROM debian

RUN apt-get update && apt-get install -y apache2 && apt-get clean && rm -rf /var/lib/apt/lists/*

ADD public_html /var/www/html/

EXPOSE 80
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Genero la imagen:

```console
docker build -t adrianjaramillo/ejemplo1:v1 .
```

Compruebo que la imagen se ha creado:

```console
atlas@olympus:~/docker/ejemplo1build/version1$ docker images
REPOSITORY                    TAG           IMAGE ID       CREATED              SIZE
adrianjaramillo/ejemplo1      v1            78598efe38b7   About a minute ago   235MB
```

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ejemplo1build/version1$ docker run -d -p 80:80 --name ejemplo1 adrianjaramillo/ejemplo1:v1
54f3831fbfba45202e6cf9019bcf6f8ca68ab557c00f573364ed55c146ec8fe2
```

Pruebo que funciona:

![version1](https://i.imgur.com/UPViWuN.png)

#### Versión 2: Desde una imagen con apache2

Muestro el `Dockerfile`:

```console
atlas@olympus:~/docker/ejemplo1build/version2$ cat Dockerfile
FROM httpd:2.4
ADD public_html /usr/local/apache2/htdocs/
EXPOSE 80
```

Genero la imagen:

```console
docker build -t adrianjaramillo/ejemplo1:v2 .
```

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ejemplo1build/version2$ docker run -d -p 80:80 --name ejemplo1 adrianjaramillo/ejemplo1:v2
c31ba3ff6a7eb6bfca1388aae222931d503f3159762c2ed1bfe387a4a977c4e6
```

Pruebo que funciona:

![version2](https://i.imgur.com/F3n5h9a.png)

#### Versión 3: Desde una imagen con nginx

Muestro el `Dockerfile`:

```console
atlas@olympus:~/docker/ejemplo1build/version3$ cat Dockerfile
FROM nginx
ADD public_html /usr/share/nginx/html
EXPOSE 80
```

Genero la imagen:

```console
docker build -t adrianjaramillo/ejemplo1:v3 .
```

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ejemplo1build/version3$ docker run -d -p 80:80 --name ejemplo1 adrianjaramillo/ejemplo1:v3
a71cf3e153588cbbc4d7b810e06a7808d2e13203fe11ab107520360abdb92ea1
```

Pruebo que funciona:

![version3](https://i.imgur.com/lFq8yaZ.png)

### Ejemplo 2: Construcción de imágenes con una aplicación PHP

#### Versión 1: Desde una imagen base sin PHP

Muestro el `Dockerfile`:

```console
FROM debian
RUN apt-get update && apt-get install -y apache2 libapache2-mod-php php && apt-get clean && rm -rf /var/lib/apt/lists/*
ADD app /var/www/html/
RUN rm /var/www/html/index.html
EXPOSE 80
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Genero la imagen:

```console
docker build -t adrianjaramillo/ejemplo2:v1 .
```

Compruebo que la imagen está:

```console
atlas@olympus:~/docker/ejemplo2build/version1$ docker images
REPOSITORY                    TAG           IMAGE ID       CREATED              SIZE
adrianjaramillo/ejemplo2      v1            4deacc77ef43   About a minute ago   254MB
```

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ejemplo2build/version1$ docker run -d -p 80:80 --name ejemplo2 adrianjaramillo/ejemplo2:v1
7629bc3a4ed786025dce58351d80599a7725353d40e0a6c5edb0cb5b7f845ebd
```

Pruebo que funciona:

![version1php](https://i.imgur.com/ZrnHwG7.png)

Muestro el `info.php`:

![version1infophp](https://i.imgur.com/loNzsGW.png)

#### Versión 2: Desde una imagen con PHP instalado

Muestro el `Dockerfile`:

```console
FROM php:7.4-apache
ADD app /var/www/html/
EXPOSE 80
```

Genero la imagen:

```console
docker build -t adrianjaramillo/ejemplo2:v2 .
```

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ejemplo2build/version2$ docker run -d -p 80:80 --name ejemplo2 adrianjaramillo/ejemplo2:v2
863fb6d27a48fbace7bf41a224ebc38db555e3cb9d9635a35506831e2268cacc
```

Pruebo que funciona:

![version2php](https://i.imgur.com/ZQ6brOt.png)

Muestro el `info.php`:

![version2infophp](https://i.imgur.com/U2IBWYf.png)

### Ejemplo 3: Construcción de imágenes con una aplicación Python

Muestro el `Dockerfile`:

```console
FROM debian
RUN apt-get update && apt-get install -y python3-pip  && apt-get clean && rm -rf /var/lib/apt/lists/*
COPY app /usr/share/app
WORKDIR /usr/share/app
RUN pip3 install --no-cache-dir -r requirements.txt
EXPOSE 3000
CMD [ "python3", "app.py"]
```

Genero la imagen:

```console
docker build -t adrianjaramillo/ejemplo3:v1 .
```

Compruebo que la imagen está:

```console
atlas@olympus:~/docker/ejemplo3build$ docker images
REPOSITORY                    TAG           IMAGE ID       CREATED          SIZE
adrianjaramillo/ejemplo3      v1            5d390f9bcd8e   47 seconds ago   510MB
```

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ejemplo3build$ docker run -d -p 80:3000 --name ejemplo3 adrianjaramillo/ejemplo3:v1
24db90990b340ce8d14bf76dc5f3f6fbf08608dc31cbb3a14b751f6c843acb42
```

Pruebo que funciona:

![ejemplo3python](https://i.imgur.com/95BJS05.png)

### Ciclo de vida de las aplicaciones

#### Paso 1: Desarrollo de nuestra aplicación

Creo una estructura de página web:

```console
atlas@olympus:~/docker/ciclovida$ mkdir public_html
atlas@olympus:~/docker/ciclovida$ cd public_html
atlas@olympus:~/docker/ciclovida/public_html$ echo "<h1>Prueba</h1>" > index.html
```

#### Paso 2: Creación de la imagen Docker

Creo el siguiente `Dockerfile`:

```console
FROM httpd:2.4
ADD ./public_html /usr/local/apache2/htdocs/
```

Genero la imagen:

```console
atlas@olympus:~/docker/ciclovida$ docker build -t adrianjaramillo/aplicacionweb:v1 .
Sending build context to Docker daemon  3.584kB
Step 1/2 : FROM httpd:2.4
 ---> acd59370d8fb
Step 2/2 : ADD ./public_html /usr/local/apache2/htdocs/
 ---> 4415b1fcea4e
Successfully built 4415b1fcea4e
Successfully tagged adrianjaramillo/aplicacionweb:v1
```

Compruebo que la imagen está:

```console
atlas@olympus:~/docker/ciclovida$ docker image ls
REPOSITORY                      TAG           IMAGE ID       CREATED              SIZE
adrianjaramillo/aplicacionweb   v1            4415b1fcea4e   About a minute ago   144MB
```

#### Paso 3: Probamos nuestra aplicación en el entorno de desarrollo

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ciclovida$ docker run --name aplweb -d -p 80:80 adrianjaramillo/aplicacionweb:v1
39c8937b5badb9dc5f910498aeaf9380d6aa1b0e326e5ea19c527ff40a499e85
```

Compruebo que esté activo:

```console
atlas@olympus:~/docker/ciclovida$ docker ps
CONTAINER ID   IMAGE                              COMMAND              CREATED              STATUS              PORTS                NAMES
39c8937b5bad   adrianjaramillo/aplicacionweb:v1   "httpd-foreground"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   aplweb
```

Pruebo que funciona:

![appdesarrollo](https://i.imgur.com/JhIrFND.png)

#### Paso 4: Distribuimos nuestra imagen

Subo a Docker Hub:

```console
atlas@olympus:~/docker/ciclovida$ docker push adrianjaramillo/aplicacionweb:v1
The push refers to repository [docker.io/adrianjaramillo/aplicacionweb]
14d158b7d9db: Pushed
2db1ee0a0f4f: Mounted from library/httpd
857d0bc0663f: Mounted from library/httpd
4a48959ac654: Mounted from library/httpd
4e2b986e3da3: Mounted from library/httpd
ad6562704f37: Mounted from library/wordpress
v1: digest: sha256:414d9c7a3d0b84fc327da8414eb9baa8abf32ce1807dae19d3448c1e0c8f84cb size: 1572
```

#### Paso 5: Implantación de la aplicación

Para simular que nuestra máquina es el entorno de producción, borramos nuestra imagen local original...

```console
atlas@olympus:~/docker/ciclovida$ docker rmi adrianjaramillo/aplicacionweb:v1
Untagged: adrianjaramillo/aplicacionweb:v1
Untagged: adrianjaramillo/aplicacionweb@sha256:414d9c7a3d0b84fc327da8414eb9baa8abf32ce1807dae19d3448c1e0c8f84cb
Deleted: sha256:4415b1fcea4ef1b5adf4e912bc4f14670ea4c713b326d16c0ca4452f57852cb0
Deleted: sha256:d2dcb5a1a671e6661428ac9e379493d6e73574bc2befa9c7027ace21ba14ed44
```

... y nos la volvemos a descargar desde Docker Hub:

```console
atlas@olympus:~/docker/ciclovida$ docker pull adrianjaramillo/aplicacionweb:v1
v1: Pulling from adrianjaramillo/aplicacionweb
42c077c10790: Already exists
77a357ba66a8: Already exists
c56c780a8904: Already exists
ecb885e1c489: Already exists
ba0337d73eed: Already exists
35e5b4af3e12: Pull complete
Digest: sha256:414d9c7a3d0b84fc327da8414eb9baa8abf32ce1807dae19d3448c1e0c8f84cb
Status: Downloaded newer image for adrianjaramillo/aplicacionweb:v1
docker.io/adrianjaramillo/aplicacionweb:v1
```

Lanzo un contenedor:

```console
tlas@olympus:~/docker/ciclovida$ docker run --name aplweb_prod -d -p 80:80 adrianjaramillo/aplicacionweb:v1
c7bf2af9a3c91426a80e22edd67d72f99a1571512d5ddd7a20f7a7dea3a39134
```

Pruebo que sigue funcionando:

![appproduccion](https://i.imgur.com/Gi0to0k.png)

#### Paso 6: Modificación de la aplicación

Si modificamos el código de la aplicación tendremos que generar una nueva imagen:

```console
atlas@olympus:~/docker/ciclovida$ cd public_html
atlas@olympus:~/docker/ciclovida/public_html$ echo "<h1>Prueba 2</h1>" > index.html
atlas@olympus:~/docker/ciclovida/public_html$ cd ..
atlas@olympus:~/docker/ciclovida$ docker build -t adrianjaramillo/aplicacionweb:v2 .
Sending build context to Docker daemon  3.584kB
Step 1/2 : FROM httpd:2.4
 ---> acd59370d8fb
Step 2/2 : ADD ./public_html /usr/local/apache2/htdocs/
 ---> 44a890a9da01
Successfully built 44a890a9da01
Successfully tagged adrianjaramillo/aplicacionweb:v2
```

Lanzo un contenedor en el entorno de desarrollo:

```console
atlas@olympus:~/docker/ciclovida$ docker run --name aplweb2 -d -p 80:80 adrianjaramillo/aplicacionweb:v2
22fc8df544b24d536aa745d502ace5215d5e56ef74365e8fb92447db881606af
```

Pruebo que funciona:

![appdesarrollo2](https://i.imgur.com/CLxrXRn.png)

Subo la nueva imagen:

```console
atlas@olympus:~/docker/ciclovida$ docker push adrianjaramillo/aplicacionweb:v2
The push refers to repository [docker.io/adrianjaramillo/aplicacionweb]
aa226b1a6620: Pushed
2db1ee0a0f4f: Layer already exists
857d0bc0663f: Layer already exists
4a48959ac654: Layer already exists
4e2b986e3da3: Layer already exists
ad6562704f37: Layer already exists
v2: digest: sha256:78c66a668b1d4b946edc1a00594480d666fb05ff4ab799d97b5e724afd3ed3ae size: 1572
```

Elimino la imagen local original:

```console
atlas@olympus:~/docker/ciclovida$ docker rmi adrianjaramillo/aplicacionweb:v2
Untagged: adrianjaramillo/aplicacionweb:v2
Untagged: adrianjaramillo/aplicacionweb@sha256:78c66a668b1d4b946edc1a00594480d666fb05ff4ab799d97b5e724afd3ed3ae
Deleted: sha256:44a890a9da01acbe029eb7ac5a57e5b2c27c114370e0a03e682739f738083f0b
Deleted: sha256:71ba067af316ee14e14780b5ce532be1bb4fc390ae803cc8e29ccff6ae05ebe0
```

La vuelvo a descargar de Docker Hub:

```console
atlas@olympus:~/docker/ciclovida$ docker pull adrianjaramillo/aplicacionweb:v2
v2: Pulling from adrianjaramillo/aplicacionweb
42c077c10790: Already exists
77a357ba66a8: Already exists
c56c780a8904: Already exists
ecb885e1c489: Already exists
ba0337d73eed: Already exists
0fb57bd0a568: Pull complete
Digest: sha256:78c66a668b1d4b946edc1a00594480d666fb05ff4ab799d97b5e724afd3ed3ae
Status: Downloaded newer image for adrianjaramillo/aplicacionweb:v2
docker.io/adrianjaramillo/aplicacionweb:v2
```

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ciclovida$ docker run --name aplweb2_prod -d -p 80:80 adrianjaramillo/aplicacionweb:v2
c9be3aaf8cb31898140aeb0967a3063debd44d2587eb01dce0dcb54a8bdb7fca
```

Pruebo que sigue funcionando:

![app2produccion](https://i.imgur.com/uz5lNat.png)

### Ejercicios creación imágenes

Muestro el directorio donde tengo la estructura de mi web y donde tendré el `Dockerfile`:

```console
atlas@olympus:~/docker/ejimagenes$ ls -la
total 12
drwxr-xr-x  3 atlas atlas 4096 Jun 17 18:04 .
drwxr-xr-x 18 atlas atlas 4096 Jun 17 17:25 ..
drwxr-xr-x  8 atlas atlas 4096 Jun 17 18:03 public_html
```

Creo el siguiente `Dockerfile`:

![dockerfilemio](https://i.imgur.com/kEXh2fG.png)

Genero la imagen:

![miimagen](https://i.imgur.com/M9rmAwR.png)

Compruebo que la tengo localmente:

```console
atlas@olympus:~/docker/ejimagenes$ docker images
REPOSITORY                        TAG           IMAGE ID       CREATED             SIZE
adrianjaramillo/mi_servidor_web   v1            c5533b4dd021   2 minutes ago       142MB
```

Lanzo un contenedor:

```console
atlas@olympus:~/docker/ejimagenes$ docker run -d -p 80:80 --name prueba1 adrianjaramillo/mi_servidor_web:v1
31196fe55dc33d3f818c4c979ba570ff6569c6fde885f1cef433d16bbd2a2f1d
```

Pruebo que funciona:

![miwebdesarrollo](https://i.imgur.com/4LlnK40.jpg)

Subo la nueva imagen a Docker Hub:

![miimagendockerhub](https://i.imgur.com/HvOgd5L.png)

Compruebo que se subió correctamente:

![miimagendockerhubprueba](https://i.imgur.com/DI6NNoF.png)

Para simular que nuestra máquina es el entorno de producción, borramos nuestra imagen local original...

```console
atlas@olympus:~/docker/ejimagenes$ docker rmi adrianjaramillo/mi_servidor_web:v1
Untagged: adrianjaramillo/mi_servidor_web:v1
Untagged: adrianjaramillo/mi_servidor_web@sha256:b1ab1fd223167e4a6b5234fa6bde88ae2e677c0cfb19b38abab12a0411ead882
Deleted: sha256:9b1fa17c1533b7f8e6596976bf1e839a9cd15e4b0f58c77008d0a62985e004c4
Deleted: sha256:a9efe2bdee3e0d17ee5316f9eec7489e1c67fa166ed7fb780de3f848ca90380d
Deleted: sha256:07a7bf31da7166779623897641084ec5f05bd7b1095a339ed89b17de8828ccb3
```

... y nos la volvemos a descargar desde Docker Hub:

![bajadademiimagen](https://i.imgur.com/nXvi5F5.png)

Lanzo otro contenedor:

![miwebfinal](https://i.imgur.com/hYpZJmA.png)

Pruebo que funciona de nuevo:

![miwebprod](https://i.imgur.com/mP96e03.jpg)
