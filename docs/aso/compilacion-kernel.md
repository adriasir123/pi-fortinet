# Compilación de Kernel

Conseguir un kernel lo más pequeño posible que tenga como mínimo:

- Teclado
- Conexión de red
- Pantalla

## Pasos previos

Dejo `/etc/apt/sources.list` de la siguiente manera:

```shell
sudo nano /etc/apt/sources.list
```

```shell
deb http://deb.debian.org/debian/ bullseye main
deb-src http://deb.debian.org/debian/ bullseye main

deb http://security.debian.org/debian-security bullseye-security main
deb-src http://security.debian.org/debian-security bullseye-security main

deb http://deb.debian.org/debian/ bullseye-updates main
deb-src http://deb.debian.org/debian/ bullseye-updates main

deb https://deb.debian.org/debian bullseye-backports main
deb-src https://deb.debian.org/debian bullseye-backports main
```

Actualizo e instalo paquetes:

```shell
sudo apt update
sudo apt install linux-source=6.0.3-1~bpo11+1 build-essential qtbase5-dev libncurses-dev
```

## Base de compilación

```shell
 atlas@olympus  ~  mkdir kernel
 atlas@olympus  ~  cd kernel
 atlas@olympus  ~/kernel  tar xf /usr/src/linux-source-6.0.tar.xz
 atlas@olympus  ~/kernel  cp /boot/config-5.10.0-18-amd64 ~/kernel/linux-source-6.0/.config
 atlas@olympus  ~/kernel 
```

## Personalización de compilación

Reduzco el `.config` actual con **solo** los módulos actualmente cargados:

```shell
make localmodconfig
```

Modifico la siguiente línea en el `Makefile` para empezar un versionado de compilaciones marcando con una tag los paquetes generados:

```shell
nano Makefile
```

```shell
EXTRAVERSION = -v1
```

Para elegir qué módulos quitar y cuáles dejar, uso el siguiente menú:

```shell
make nconfig
```

## Compilación

Podemos hacerla de la manera simple:

```shell
make -j 12 bindeb-pkg
```

O de la manera completa:

```shell
time make -j 12 bindeb-pkg && ( speaker-test -t sine -f 500 )& pid=$! ; sleep 0.5s ; kill -9 $pid # (1)!
```

1. Obtendremos la duración completa de ejecución al finalizar y un pitido por los altavoces de aviso


Nos mostrará 

Si queremos controlar el tiempo de compilación, ejecutamos de la siguiente manera:







Los ficheros deb generados se encontrarán en el directorio padre:

```shell
 atlas@olympus  ~/kernel/linux-source-6.0  cd ..
 atlas@olympus  ~/kernel  ls
.rw-r--r-- atlas atlas  15 MB Thu Nov 10 13:40:59 2022  linux-image-6.0.3-v1_6.0.3-v1-1_amd64.deb
.rw-r--r-- atlas atlas 8.1 MB Thu Nov 10 13:40:32 2022  linux-headers-6.0.3-v1_6.0.3-v1-1_amd64.deb
.rw-r--r-- atlas atlas 1.2 MB Thu Nov 10 13:40:46 2022  linux-libc-dev_6.0.3-v1-1_amd64.deb
.rw-r--r-- atlas atlas 5.1 KB Thu Nov 10 13:41:00 2022  linux-upstream_6.0.3-v1-1_amd64.buildinfo
drwxr-xr-x atlas atlas 4.0 KB Thu Nov 10 13:41:54 2022  ..
drwxr-xr-x atlas atlas 4.0 KB Thu Nov 10 13:40:20 2022  linux-source-6.0
.rw-r--r-- atlas atlas 1.8 KB Thu Nov 10 13:41:00 2022  linux-upstream_6.0.3-v1-1_amd64.changes
drwxr-xr-x atlas atlas 269 B  Thu Nov 10 13:41:00 2022  .
```

make -j12 clean para hacer una nueva compilación

## Instalación

```shell
sudo dpkg -i linux-image-6.0.3-v1_6.0.3-v1-1_amd64.deb
```

Desinstalación

sudo dpkg -P linux-image-4.19.67-compile_4.19.67-compile-9_amd64.deb





## Proceso de reducción

Cuando una compilación funcione, me copio el .config en el directorio padre:

```shell
cp .config ../.configv1
```












## Entregas


entrega finalmente el fichero deb con el kérnel compilado por ti.


make clean

Tamaño en bytes de vmlinuz: 3502640






cat .config | grep "=y" | wc -l

cat .config | grep "=m" | wc -l




[]()









https://github.com/adriasir123/kernel-config-backups






cat .config | grep "=y" | wc -l



sudo dpkg -i linux-image

make -j 12 clean



## Conclusiones



