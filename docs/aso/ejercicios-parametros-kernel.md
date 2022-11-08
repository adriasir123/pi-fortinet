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

> Deshabilitar si es posible el Kernel Mode Setting (KMS) de la tarjeta gráfica

Compruebo que tengo KMS activado:

```shell
vagrant@parametroskernel:~$ lsmod | grep kms
drm_kms_helper        278528  3 cirrus
cec                    61440  1 drm_kms_helper
drm                   626688  3 drm_kms_helper,cirrus
```

Modifico la siguiente línea de `/etc/default/grub` así:

```shell
GRUB_CMDLINE_LINUX_DEFAULT="net.ifnames=0 biosdevname=0 nomodeset" # (1)
```

1. Añado nomodeset al final




    Cambia provisionalmente la swappiness para que la swap de tu equipo se active cuando se use más de un 90% de la RAM.
    Haz que el cambio de la swappiness sea permanente.
    Muestra el valor del bit de forward para IPv6.
    Deshabilita completamente las Magic Sysrq en el arranque y vuelve a habilitarlas después de reiniciar.





