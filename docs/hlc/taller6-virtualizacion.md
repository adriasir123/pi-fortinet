# Taller 6: Gestión de redes en QEMU/KVM + libvirt

La teoría para completar este taller se encuentra en el [capítulo 7 del curso de virtualización](https://github.com/josedom24/curso_virtualizacion_linux).

test github pages

test github pages 2

test github pages 3

test github pages 4

## Entrega

### Parte 1

> Mostrar el comando usado para clonar `debian11`

```shell
virt-clone --connect=qemu:///system --original debian11 --name maquina-clonada --file /var/lib/libvirt/images/maquina-clonada.qcow2
```

> ¿Qué cambios has hecho en la nueva máquina para que no sea igual a la original?

He cambiado el hostname.

Muestro que no cambió en la clonación, y que lo cambiamos:

![cambiohostname](https://i.imgur.com/naWLekl.png)

Tras un reinicio, el cambio será efectivo:

![reiniciohostname](https://i.imgur.com/b6secb7.png)










































## Desarrollo

### Ejercicio 1

> Clonar la VM `debian11` con `virt-clone` y nombre `maquina-clonada`

```shell
atlas@olympus:~$ virt-clone --connect=qemu:///system --original debian11 --name maquina-clonada --file /var/lib/libvirt/images/maquina-clonada.qcow2
Allocating 'maquina-clonada.qcow2'                                                                                                                     |  20 GB  00:00:09

Clone 'maquina-clonada' created successfully.
```

> Cambiar el hostname antiguo en `maquina-clonada`

Muestro que el hostname no cambió en la clonación, y que lo cambiamos:

![cambiohostname](https://i.imgur.com/naWLekl.png)

Tras un reinicio, el cambio será efectivo:

![reiniciohostname](https://i.imgur.com/b6secb7.png)