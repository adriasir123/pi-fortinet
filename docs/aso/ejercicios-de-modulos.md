# Ejercicios de manejo de módulos

## Ejercicio 1

> Listar los módulos cargados

Podemos hacer:

```shell
lsmod
```

[Output](https://gist.github.com/adriasir123/b3232f0a2e96adc0405718a26757de43)

O también:

```shell
cat /proc/modules
```

[Output](https://gist.github.com/adriasir123/3d8b114c08aba31f489fb1cbecebd3db)

## Ejercicio 2

> Contar el número de módulos disponibles en el núcleo

```shell
 atlas@olympus  ~  find /lib/modules/$(uname -r)/kernel -type f -iname '*.ko' | wc -l
3900
```

## Ejercicio 3

> Monitorizar con `sudo dmesg -wH` la conexión de un pendrive

```shell
sudo dmesg -wH
```

Muestro los cambios:

![dmesgpendrive](https://i.postimg.cc/Dw2J1tHH/dmesgusb.gif)

## Ejercicio 4

> Elegir un módulo no esencial

Elijo `kvm_intel`

> Comprobar que está actualmente cargado

```shell
 atlas@olympus  ~  lsmod | grep -w 'kvm_intel'
kvm_intel             331776  0
kvm                   937984  1 kvm_intel
```

> Descargar el módulo

```shell
sudo modprobe -r kvm_intel
```

> Comprobar qué ocurre

Ya no nos aparece en el listado:

```shell
 atlas@olympus  ~  lsmod | grep -w 'kvm_intel'
 ✘ atlas@olympus  ~ 
```

Además, como es lógico, KVM dejará de funcionar:

![kvmsinfuncionar](https://i.imgur.com/ymImfRA.png)

> Volver a cargarlo

```shell
sudo modprobe kvm_intel
```

Compruebo que vuelve a aparecer en el listado:

```shell
 atlas@olympus  ~  lsmod | grep -w 'kvm_intel'
kvm_intel             331776  0
kvm                   937984  1 kvm_intel
```

KVM volverá a funcionar:

![kvmfuncionando](https://i.imgur.com/UNgHHw1.png)

## Ejercicio 5

> Elegir un módulo que esté en uso

Elijo `kvm_intel` y compruebo que está cargado:

```shell
 atlas@olympus  ~  lsmod | grep -w 'kvm_intel'
kvm_intel             331776  0
kvm                   937984  1 kvm_intel
```

> Configurar el arranque para que no se cargue automáticamente

Creo el siguiente fichero:

```shell
sudo nano /etc/modprobe.d/kvm_intel.conf
```

Con los contenidos:

```shell
blacklist kvm_intel
```

Ejecuto:

```shell
sudo depmod -ae
sudo update-initramfs -u
```

Tras reiniciar, por ejemplo, no puedo iniciar máquinas de KVM:

![kvmsinfuncionar](https://i.imgur.com/ymImfRA.png)

## Ejercicio 6

> Cargar el módulo loop

```shell
sudo modprobe loop
```

Compruebo que se ha cargado:

```shell
 atlas@olympus  ~  lsmod | grep loop
loop                   40960  0
```

> Obtener información

```shell
 atlas@olympus  ~  sudo modinfo loop
filename:       /lib/modules/5.10.0-19-amd64/kernel/drivers/block/loop.ko
alias:          devname:loop-control
alias:          char-major-10-237
alias:          block-major-7-*
license:        GPL
depends:
retpoline:      Y
intree:         Y
name:           loop
vermagic:       5.10.0-19-amd64 SMP mod_unload modversions
sig_id:         PKCS#7
signer:         Debian Secure Boot CA
sig_key:        32:A0:28:7F:84:1A:03:6F:A3:93:C1:E0:65:C4:3A:E6:B2:42:26:43
sig_hashalgo:   sha256
signature:      72:B9:28:99:6F:56:69:20:25:13:90:20:7B:3A:2B:77:AE:2B:44:14:
	        C1:3D:83:4B:8E:4D:85:30:32:82:D6:62:31:79:80:F3:F8:C7:65:1E:
	        26:BF:89:AA:F6:DF:E2:E5:45:55:35:D3:D6:CE:79:40:77:A2:BF:29:
	        53:C1:33:3E:80:2B:D6:F6:86:C4:78:D6:2B:27:7A:B6:65:62:66:B8:
	        9D:FE:A3:F5:FF:01:2E:20:E0:6A:65:C3:98:17:44:58:CA:F8:D7:1F:
	        D9:D4:59:C8:03:A1:25:2C:76:EE:5B:5C:C4:E4:36:3F:1E:06:AD:07:
	        76:F4:08:6A:B1:E1:37:32:5C:53:86:96:62:6F:D3:29:33:DE:26:C0:
	        9F:7D:95:2A:FB:AB:F8:C7:80:B9:7F:88:DA:3B:26:9F:FB:E7:D4:E5:
	        76:48:DE:66:B3:17:0F:C9:D5:EE:56:45:9F:CC:C7:A6:9F:9C:2E:F8:
	        41:01:83:A2:AD:20:E7:CD:31:B1:13:A7:12:E6:65:E0:B9:A7:B0:7A:
	        16:E5:34:1F:D6:FE:BF:76:6F:7E:95:12:00:45:8A:80:9A:13:EA:80:
	        CF:AB:5D:7D:E2:01:93:F8:9F:C2:8E:6C:6C:BD:ED:DF:86:FE:17:B0:
	        70:B2:66:E9:3E:E1:7A:ED:73:C8:5B:D6:7A:78:6D:3D
parm:           max_loop:Maximum number of loop devices (int)
parm:           max_part:Maximum number of partitions per loop device (int)
```

> Explicar para qué sirve

Es el módulo encargado de controlar los loop devices, que son pseudo-dispositivos que posibilitan el acceso a determinados ficheros como si fueran dispositivos de bloque.

Los podemos listar de la siguiente manera:

```shell
 atlas@olympus  ~  ls /dev/loop*
brw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022 ﰩ /dev/loop0
brw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022 ﰩ /dev/loop1
brw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022 ﰩ /dev/loop2
brw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022 ﰩ /dev/loop3
brw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022 ﰩ /dev/loop4
brw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022 ﰩ /dev/loop5
brw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022 ﰩ /dev/loop6
brw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022 ﰩ /dev/loop7
crw-rw---- root disk 0 B Thu Nov  3 17:14:53 2022  /dev/loop-control
```

> Listar el contenido de /sys/module/loop/parameters

```shell
 atlas@olympus  ~  ls /sys/module/loop/parameters
.r--r--r-- root root 4.0 KB Thu Nov  3 18:26:35 2022  max_loop
.r--r--r-- root root 4.0 KB Thu Nov  3 18:26:35 2022  max_part
drwxr-xr-x root root   0 B  Thu Nov  3 18:26:34 2022  .
drwxr-xr-x root root   0 B  Thu Nov  3 17:14:53 2022  ..
```

> Configurar el módulo para tener como máximo 12 dispositivos loop

Creo el siguiente fichero:

```shell
sudo nano /etc/modprobe.d/loop.conf
```

Con el siguiente contenido:

```shell
options loop max_loop=12
```

Si recargamos el módulo y volvemos a listar nuestros dispositivos loop, veremos que ahora tenemos 12:

```shell
 atlas@olympus  ~  sudo modprobe -r loop
 atlas@olympus  ~  sudo modprobe loop
 atlas@olympus  ~  ls /dev/loop*
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop0
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop1
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop10
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop11
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop2
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop3
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop4
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop5
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop6
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop7
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop8
brw-rw---- root disk 0 B Thu Nov  3 18:43:57 2022 ﰩ /dev/loop9
crw-rw---- root disk 0 B Thu Nov  3 18:43:56 2022  /dev/loop-control
```
