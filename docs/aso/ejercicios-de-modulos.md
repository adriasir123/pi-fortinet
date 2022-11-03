# Ejercicios de manejo de módulos

## Ejercicio 1

> Listar los módulos cargados

Podemos hacer:

```shell
lsmod
```

O también:

```shell
less /proc/modules
```

[Output](https://gist.github.com/adriasir123/b3232f0a2e96adc0405718a26757de43)

## Ejercicio 2

> Contar el número de módulos disponibles en el núcleo que estás usando

```shell
 atlas@olympus  ~  find /lib/modules/$(uname -r)/kernel -type f -iname '*.ko' | wc -l
3900
```

## Ejercicio 3

> Conectar USB y observa la salida de la instrucción sudo dmesg

```shell
sudo dmesg -wH
```

GIF para que se vean los cambios:

![gifdmesg](https://i.postimg.cc/Dw2J1tHH/dmesgusb.gif)

## Ejercicio 4

> Elegir un módulo no esencial

Elijo el módulo `lp`.

> Comprobar que está actualmente cargado

```shell
 atlas@olympus  ~  lsmod | grep -w 'lp'
lp                     20480  0
```

> Eliminar el módulo

```shell
sudo modprobe -r lp
```

> Comprobar qué ocurre

```shell
 atlas@olympus  ~  lsmod | grep -w 'lp'
 ✘ atlas@olympus  ~ 
```

Ya no nos aparece en el listado.

> Volver a cargarlo

```shell
sudo modprobe lp
```

Compruebo que vuelve a aparecer en el listado:

```shell
 atlas@olympus  ~  lsmod | grep -w 'lp'
lp                     20480  0
```

## Ejercicio 5

> Selecciona un módulo que esté en uso en tu equipo y configura el arranque para que no se cargue automáticamente

Creo el fichero `/etc/modprobe.d/kvm_intel.conf` y `/etc/modprobe.d/kvm.conf`:

```shell
sudo nano /etc/modprobe.d/kvm_intel.conf
sudo nano /etc/modprobe.d/kvm.conf
```

Con los contenidos:

```shell
blacklist kvm_intel
```

Y:

```shell
blacklist kvm
```

Ejecuto:

```shell
sudo depmod -ae
sudo update-initramfs -u
```

Tras reiniciar, no puedo inicar máquinas de kvm por ejemplo:

![nofuncionakvm](https://i.imgur.com/ymImfRA.png)

## Ejercicio 6

> Carga el módulo loop, obtén información de qué es y para qué sirve. Lista el contenido de /sys/modules/loop/parameters y configura el equipo para que se puedan cargar como máximo 12 dispositvos loop la próxima vez que se arranque.



















