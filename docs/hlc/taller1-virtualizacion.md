# Taller 1: Instalación de QEMU/libvirt. Conexión local y remota

La teoría para completar este taller se encuentra en el [capítulo 2 del curso de virtualización](https://github.com/josedom24/curso_virtualizacion_linux).

## Entrega

### Parte 1

> Entregar la salida de `virsh version` en tu máquina

```shell
atlas@olympus:~$ virsh version
Compiled against library: libvirt 7.0.0
Using library: libvirt 7.0.0
Using API: QEMU 7.0.0
Running hypervisor: QEMU 5.2.0
```

### Parte 2

> Ejecutar `virsh list` usando `qemu:///system`

```shell
atlas@olympus:~$ virsh -c qemu:///system list
 Id   Name      State
-------------------------
 4    preseed   running
```

### Parte 3

> ¿Por qué al ejecutar `virsh -c qemu:///system list --all` no aparece la máquina creada por gnome-boxes?

Muestro mi salida del comando:

```shell
atlas@olympus:~$ virsh -c qemu:///system list --all
 Id   Name              State
----------------------------------
 4    preseed           running
 -    pruebas_pruebas   shut off
```

No aparece porque gnome-boxes usa `qemu:///session` en vez de `qemu:///system`, y son 2 sesiones totalmente diferenciadas.

Para curiosos, más info [aquí](https://blog.wikichoon.com/2016/01/qemusystem-vs-qemusession.html).

### Parte 4

> Mostrar la salida del comando virsh que muestra la máquina de gnome-boxes

```shell
atlas@olympus:~$ virsh -c qemu:///session list
 Id   Name          State
-----------------------------
 1    ubuntu20.10   running
```

### Parte 5

> Explicar cómo configurar tu equipo para realizar conexiones remotas de libvirt

Si queremos hacer una conexión remota, por ejemplo, al libvirt de Roberto, él tendrá que insertar mi clave pública SSH en su `.ssh/authorized_keys`:

```shell
roberto@portatil:~/.ssh$ cat authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDqPHr0Q41RPMvMXDEYoevcCp+DPtzHLefK35Azea7iH/okOv4IlD3Yf1c/HEJNX/eTP/QxJwl8rQtoJ78Oe69FKKrLujaBl1RmmTXJMAg30zLay2mTiiANikyeu2W+dcuZAycDxTb8vUpAJheF16cH446+ZgYLPh5XafsH00ALtX6Abu8GQhJgoRMYuyHvM6BBjQxRdqmSBHZdpds+raCRrfHf7vFg4umGOPHAKmd5WIn7EULVEMX67R+t/pPfLMkAM7OX8/NLSlMADsqmI7TjoJiFuELwY/usWRodXCxq8iS4pDgldM7zx1gfIe+C5Upz7CtSpJaNNnr2gTTCCQg/Oz230Lh5DEWN0zxtz8iBMwV6uOj9baWkOTxxu9Lk95g6QIH1oFRsJDEmAvcfPIvUcvz2i0P8X5wglbzDQFSdH3UR0ndqsImXk8FtNWO/iHw0LThGh0t+RKUV+Y6h/nQdFIzkiKXqOsVRARx66zT8VgCBy3p6O6NpnxtyZ6VD6sc= atlas@olympus
```

De la misma manera, si alguien quisiera hacer conexiones remotas a mi libvirt, yo tendría que añadir su clave pública SSH a mi `.ssh/authorized_keys`.

> Entregar la salida de `virsh` haciendo una conexión remota a Roberto

Pruebo a listar sus redes remotas:

```shell
atlas@olympus:~$ virsh -c qemu+ssh://roberto@192.168.121.74/system net-list --all
 Name      State      Autostart   Persistent
----------------------------------------------
 default   inactive   no          yes
```

## Desarrollo

### Ejercicio 1

> Instalar QEMU/KVM + libvirt

```shell
sudo apt install qemu-system libvirt-clients libvirt-daemon-system
```

Compruebo que se ha instalado:

```shell
vagrant@pruebas:~$ virsh version
Compiled against library: libvirt 7.0.0
Using library: libvirt 7.0.0
Using API: QEMU 7.0.0
Running hypervisor: QEMU 5.2.0
```

### Ejercicio 2

> Configurar el usuario sin privilegios para que utilice `qemu:///system` sin root

Añado el usuario `vagrant` al grupo `libvirt`:

```shell
sudo usermod -a -G libvirt vagrant
```

Pruebo que funciona:

```shell
vagrant@pruebas:~$ virsh -c qemu:///system net-list --all
 Name      State      Autostart   Persistent
----------------------------------------------
 default   inactive   no          yes
```

### Ejercicio 3

> Instalar `gnome-boxes` con una máquina virtual

```shell
sudo apt install gnome-boxes
```

![boxesvm1](https://i.imgur.com/xuop45h.png)

![boxesvm2](https://i.imgur.com/D70HQ36.png)

![boxesvm3](https://i.imgur.com/yqHq1Hh.png)

![boxesvm4](https://i.imgur.com/JX6NBZL.png)

![boxesvm5](https://i.imgur.com/ZYqxkf6.png)

![boxesvm6](https://i.imgur.com/KfG6wDY.png)

### Ejercicio 4

> Conectar remotamente al libvirt de Roberto

Roberto tiene que insertar mi clave pública SSH en su `.ssh/authorized_keys`:

```shell
roberto@portatil:~/.ssh$ cat authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDqPHr0Q41RPMvMXDEYoevcCp+DPtzHLefK35Azea7iH/okOv4IlD3Yf1c/HEJNX/eTP/QxJwl8rQtoJ78Oe69FKKrLujaBl1RmmTXJMAg30zLay2mTiiANikyeu2W+dcuZAycDxTb8vUpAJheF16cH446+ZgYLPh5XafsH00ALtX6Abu8GQhJgoRMYuyHvM6BBjQxRdqmSBHZdpds+raCRrfHf7vFg4umGOPHAKmd5WIn7EULVEMX67R+t/pPfLMkAM7OX8/NLSlMADsqmI7TjoJiFuELwY/usWRodXCxq8iS4pDgldM7zx1gfIe+C5Upz7CtSpJaNNnr2gTTCCQg/Oz230Lh5DEWN0zxtz8iBMwV6uOj9baWkOTxxu9Lk95g6QIH1oFRsJDEmAvcfPIvUcvz2i0P8X5wglbzDQFSdH3UR0ndqsImXk8FtNWO/iHw0LThGh0t+RKUV+Y6h/nQdFIzkiKXqOsVRARx66zT8VgCBy3p6O6NpnxtyZ6VD6sc= atlas@olympus
```

Pruebo a listar las redes remotas de Roberto:

```shell
atlas@olympus:~$ virsh -c qemu+ssh://roberto@192.168.121.74/system net-list --all
 Name      State      Autostart   Persistent
----------------------------------------------
 default   inactive   no          yes
```

> Configurar tu máquina para que Roberto pueda acceder remotamente a tu libvirt

Simplemente tendríamos que añadir su clave pública SSH a mi `.ssh/authorized_keys`:

```shell
atlas@olympus:~/.ssh$ cat authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDdXLqFJcBCixl8EAWwNoVgN5qKKGaFN5g0iMg6im+Tlbc31hvtBkbkXWYg5YvB981RCmfBjO2ZS2j95wqRTKBEJijdqYg4oiezwLMN1VWtHhXtggUXSKPa8HGIetz9R2rYAmsrRaFWTcyxX5xsXGESMWbesxZm3A/WFVneDmN+dcLd09sEGW2HB/wFgT6/kmzT0Vd5IkfwKhvcFNhu2QlHNY9eoZ02l7eYkCrIokd1ZzA2AHLwFXPfJLrKQ7feRF5EETL/dFdcV1tuZZZmRm8AYZIiZRyBaqMu/UqR6l+VSx3XONlVG1Ug2xo/2RuDyDQ2i+QGFo6ZjjbmnKVXg/Vil+bVUQGa/g3sOnbAgC2N0hmZ5T4Pi7iAAtMGltrWf2VpmzsZUFd3SJ6IET3SLd5R9l4PknNuzDY5ROIhNLarmkbplfzqT5+V14dOFiFXmLAFimCtI3Jd+tllQhH5P0n7Whg9QonUh5OgwchsLtniovC4R1ccoB7jzWxpvl1BT78= roberto@portatil
```
