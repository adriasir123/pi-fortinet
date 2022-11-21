# Migración CentOS

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
deb https://deb.debian.org/debian/ bullseye main
deb-src https://deb.debian.org/debian/ bullseye main

deb https://security.debian.org/debian-security bullseye-security main
deb-src https://security.debian.org/debian-security bullseye-security main

deb https://deb.debian.org/debian/ bullseye-updates main
deb-src https://deb.debian.org/debian/ bullseye-updates main

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
time make -j 12 bindeb-pkg && ( speaker-test -t sine -f 500 )& pid=$! ; sleep 0.5s ; kill -9 $pid
```

!!! info

    Al finalizar, con la manera completa, obtendremos la duración de la compilación  y un pitido por los altavoces de aviso

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

## Instalación

```shell
sudo dpkg -i linux-image-6.0.3-v1_6.0.3-v1-1_amd64.deb
```

Los contenidos estarán en `/boot`

Para desinstalar:

```shell
sudo dpkg -P linux-image-6.0.3-v1
```

También podríamos desinstalar borrando manualmente los ficheros de `/boot` que no necesitemos

## Proceso de reducción

Tras instalar un kernel, reiniciar la máquina, y cargarlo desde la vista avanzada de grub, pueden pasar 2 cosas:

- Que funcione
- Que no funcione

### Si funciona

Hago una copia de seguridad versionada del `.config`:

```shell
cp .config ../kernel-config-backups/.configv1
```

Subo el `EXTRAVERSION` del `Makefile` de número

Limpio los restos de la anterior compilación:

```shell
make -j 12 clean
```

Compilo de nuevo

### Si no funciona

Recupero la copia de seguridad más actualizada del `.config` que tenga en ese momento:

```shell
cp ../kernel-config-backups/.configv1 .config
```

Limpio los restos de la anterior compilación:

```shell
make -j 12 clean
```

Compilo de nuevo

## Entregas

Muestro la cantidad de módulos estáticos y dinámicos:

```shell
cat .config | grep "=y" | wc -l
cat .config | grep "=m" | wc -l
```

![conteofinal](https://i.imgur.com/s5YpfzS.png)

Repositorio con el historial de compilaciones:

<https://github.com/adriasir123/kernel-config-backups>

Comparación de tamaño de los vmlinuz:

![vmlinuzs](https://i.imgur.com/5j8T06X.png)

El tamaño se ha reducido a la mitad

Aquí puedes descargarte el paquete final:

<https://www.mediafire.com/file/8taqlicfk2eyos0/linux-image-6.0.3-vfinal_6.0.3-vfinal-147_amd64.deb/file>

## Conclusiones

En esta práctica se aprende mucho sobre la relación tan directa que hay entre el kernel y el hardware de la máquina, pero tiene graves problemas que se deberían de solucionar:

- Llega un momento en que **se vuelve repetitiva y ya no aprendes nada nuevo**, sólo estás haciendo las cosas de forma mecánica
- **No es para nada igualitaria, genera desigualdad**. Este es sin duda el peor problema. Cada persona tiene un portátil con características diferentes, y no es lo mismo alguien con un procesador i7 que alguien con un i3. Hay gente que ha tenido compilaciones de 1 hora, y otra gente como yo que empecé teniendo compilaciones de 12 minutos cuando todavía no había empezado a quitar módulos

Soluciones que propongo:

- En lugar de que el objetivo de la práctica sea conseguir el kernel más pequeño posible, cambiarlo a que el objetivo sea llegar a un número de compilaciones razonables, por ejemplo 10-20
- Utilizar OpenStack para las compilaciones, así se eliminaría el problema de la desigualdad que genera la práctica. Es cierto que al ser máquinas virtuales no es la misma experiencia porque el hardware es virtualizado, pero al final el proceso de compilación es el mismo, y que el hardware sea virtualizado no creo que genere un problema a la hora de aprender
