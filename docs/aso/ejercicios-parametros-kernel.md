# Ejercicios de parámetros del kernel

## Ejercicio 1

> Deshabilitar apparmor

Muestro el estado de apparmor:

```shell
vagrant@parametroskernel:~$ cat /sys/module/apparmor/parameters/enabled
Y
```

Y = habilitado

Para deshabilitarlo:

```shell
echo 'GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT apparmor=0"' | sudo tee /etc/default/grub.d/apparmor.cfg
sudo update-grub
sudo reboot
```

Compruebo que se ha deshabilitado:

```shell
vagrant@parametroskernel:~$ cat /sys/module/apparmor/parameters/enabled
N
```

N = deshabilitado

## Ejercicio 2

> Deshabilitar el Kernel Mode Setting (KMS) de la tarjeta gráfica

Compruebo que tengo KMS activado:

```shell
vagrant@parametroskernel:~$ lsmod | grep kms
drm_kms_helper        278528  3 cirrus
cec                    61440  1 drm_kms_helper
drm                   626688  3 drm_kms_helper,cirrus
```

Modifico la siguiente línea de `/etc/default/grub`:

```shell
GRUB_CMDLINE_LINUX_DEFAULT="net.ifnames=0 biosdevname=0 nomodeset" # (1)!
```

1. Añado nomodeset al final

Actualizo grub:

```shell
sudo update-grub
```

Reinicio:

```shell
sudo reboot
```

Sé que se ha deshabilitado correctamente porque al reiniciar tengo la resolución mal:

![kmsdesactivado](https://i.imgur.com/gSHqjPV.png)

Y además también estaba mal la resolución durante el arranque.

## Ejercicio 3

> Modificar temporalmente la swappiness para que la swap se active solamente cuando la carga de RAM supere el 90%

Muestro el valor actual:

```shell
 atlas@olympus  ~  sudo sysctl vm.swappiness
vm.swappiness = 60
```

Lo modifico:

```shell
sudo bash -c "echo 10 > /proc/sys/vm/swappiness"
```

Compruebo que el valor ha cambiado:

```shell
 atlas@olympus  ~  sudo sysctl vm.swappiness
vm.swappiness = 10
```

Para testear el funcionamiento tengo que instalar el siguiente paquete:

```shell
sudo apt install stress-ng
```

Mi máquina tiene 7.6 Gi de RAM así que la SWAP se debería de activar aproximadamente con 6.84 Gi de RAM en uso.

Abro 2 terminales, en la primera ejecuto el siguiente comando para monitorizar la  memoria:

```shell
watch free -h
```

En la segunda ejecuto el siguiente comando para llenar la memoria:

```shell
stress-ng --vm 1 --vm-bytes 120%
```

Los porcentajes no son muy exactos, pero lo importante es que llenemos la RAM (casi toda) lo suficiente para que empiece a usar la SWAP.

GIF de funcionamiento:

![swap90temporal](https://i.postimg.cc/qqxzJtGQ/swap90temporal.gif)

Si queremos reiniciar la prueba por algún motivo, como cambiar las cargas de RAM, tendremos que recargar la SWAP:

```shell
sudo swapoff -a
sudo swapon -a
```

## Ejercicio 4

> Haz que el cambio de la swappiness sea permanente








    Muestra el valor del bit de forward para IPv6.
    Deshabilita completamente las Magic Sysrq en el arranque y vuelve a habilitarlas después de reiniciar.





