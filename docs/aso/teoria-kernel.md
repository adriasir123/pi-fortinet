# Teoría

## Configuración y compilación del núcleo linux 1

<https://www.youtube.com/watch?v=xMYq2nae-94>

<iframe width="1252" height="704" src="https://www.youtube.com/embed/xMYq2nae-94" title="Configuración y compilación del núcleo linux (1)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```shell
sudo apt update
sudo apt search linux-source
```

En `bullseye-backports` tenemos la versión 6.0, añadir los repos:

```shell
deb https://deb.debian.org/debian bullseye-backports main
deb-src https://deb.debian.org/debian bullseye-backports main
```

Kernel actual:

```shell
uname -r
```

Lista de código fuente disponible para instalar:

```shell
sudo apt policy linux-source
```

Instalo los paquetes necesarios:

```shell
sudo apt install linux-source=6.0.3-1~bpo11+1 build-essential qtbase5-dev
```

Compruebo las fuentes instaladas:

```shell
dpkg -l | grep linux-source
```

Contenidos del paquete:

```shell
dpkg -L linux-source-6.0
```

Las fuentes se descargan en `/usr/src`:

```shell
vagrant@kernel:~$ ls -la /usr/src
total 131004
drwxr-xr-x  3 root root      4096 Nov  7 22:56 .
drwxr-xr-x 14 root root      4096 Sep 12 05:16 ..
drwxr-xr-x  2 root root      4096 Nov  7 22:56 linux-config-6.0
-rw-r--r--  1 root root     55312 Oct 29 15:59 linux-patch-6.0-rt.patch.xz
-rw-r--r--  1 root root 134077800 Oct 29 15:59 linux-source-6.0.tar.xz
```

Descomprimo las fuentes en un directorio del home del usuario:

```shell
vagrant@kernel:~$ mkdir kernel
vagrant@kernel:~$ cd kernel
vagrant@kernel:~/kernel$ tar xf /usr/src/linux-source-6.0.tar.xz
vagrant@kernel:~/kernel$ ls -la
total 12
drwxr-xr-x  3 vagrant vagrant 4096 Nov  7 23:55 .
drwxr-xr-x  5 vagrant vagrant 4096 Nov  7 23:55 ..
drwxr-xr-x 25 vagrant vagrant 4096 Oct 29 15:59 linux-source-6.0
```

Borrar datos de una compilación anterior:

```shell
make clean
```

Limpieza profunda:

```shell
make mrproper
```

Fichero config para la compilación del que se parte:

```shell
/boot/config-5.10.0-18-amd64
```

=y estático

=m dinámico

not set

Me copio ese fichero config como punto de partida a mi directorio de compilación:

```shell
cp /boot/config-5.10.0-18-amd64 ~/kernel/linux-source-6.0/.config
```

Dejo el config en un estado similar a como está la máquina actualmente:

```shell
make oldconfig
```

Esta opción seguramente no se usa y además creo recordar que dejaba los módulos no cargados también. La mejor opción es:

```shell
make localmodconfig
```

Compilar generando un paquete deb con sólo el binario:

```shell
make -j5 bindeb-pkg
```

Si no se especifica el número con -j coge todos los recursos que puede, y puede ser problemático llevando a errores, siempre especificar cuántos cores.

Al terminar la compilación tenemos los siguientes mensajes de información:

```shell
dpkg-deb: building package 'linux-libc-dev' in '../linux-libc-dev_6.0.3-1_amd64.deb'.
dpkg-deb: building package 'linux-image-6.0.3' in '../linux-image-6.0.3_6.0.3-1_amd64.deb'.
 dpkg-genbuildinfo --build=binary
 dpkg-genchanges --build=binary >../linux-upstream_6.0.3-1_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .
dpkg-buildpackage: info: binary-only upload (no source included)
```

Nos están indicando que los paquetes deb que necesitemos se encuentran en el directorio padre:

```shell
vagrant@kernel:~/kernel$ ls -la
total 18276
drwxr-xr-x  3 vagrant vagrant    4096 Nov  8 01:08 .
drwxr-xr-x  5 vagrant vagrant    4096 Nov  7 23:55 ..
-rw-r--r--  1 vagrant vagrant 8486908 Nov  8 01:07 linux-headers-6.0.3_6.0.3-1_amd64.deb
-rw-r--r--  1 vagrant vagrant 8930780 Nov  8 01:08 linux-image-6.0.3_6.0.3-1_amd64.deb
-rw-r--r--  1 vagrant vagrant 1266908 Nov  8 01:08 linux-libc-dev_6.0.3-1_amd64.deb
drwxr-xr-x 29 vagrant vagrant    4096 Nov  8 01:07 linux-source-6.0
-rw-r--r--  1 vagrant vagrant    5115 Nov  8 01:08 linux-upstream_6.0.3-1_amd64.buildinfo
-rw-r--r--  1 vagrant vagrant    1761 Nov  8 01:08 linux-upstream_6.0.3-1_amd64.changes
```

## Configuración y compilación del núcleo linux 2

<https://www.youtube.com/watch?v=iDgtcLsI5FU>

Conteo de módulos:

```shell
cat .config | grep "=y" | wc -l

cat .config | grep "=m" | wc -l
```

## 2/11/22 Módulos

Enlazado dinámico

/lib/modules/$(uname -r)/kernel

/usr/lib/modules

lsmod en la salida hay 0 o 3 o 4, quiere decir los módulos que dependen de tal módulo, son dependencias

modprobe -r (quitar)

modprobe nombre

modinfo

vmlinuz es el enlazado estático

dkms

## Misc

<http://kernel.org>

EOL (end of life no hay más soporte)

apt policy linux-image-amd64

vmlinuz son los kernel

/boot están los kernel (fichero config y vmlinuz

efitbootmgr (cómo está el arranque en la bios‌

Podemos compilar en una máquina virtual para hacer pruebas, lo de verdad en la real

‌

sudo apt install linux-image-amd64

‌

Para seleccionar el kernel se hace en el grub

‌

uname -r para comprobar el kernel que tenemos

‌

sudo apt purge linux-image-amd64 (desinstalación limpia)

‌

sudo rm -r /boot/vmlinuz-5.19.0.2-amd64 initrd y el config

sudo update-grub (actualizar las entradas)‌
