# Taller 4: Gestión del almacenamiento en QEMU/KVM + libvirt

La teoría para completar este taller se encuentra en el [capítulo 5 del curso de virtualización](https://github.com/josedom24/curso_virtualizacion_linux).

## Entrega

### Parte 1

> Mostrar los pools con `virsh`

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-list
 Name      State    Autostart
-------------------------------
 default   active   yes
 isos      active   yes
```

> ¿De qué tipo son?

`dir`, y lo demuestro así:

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-list --type dir
 Name      State    Autostart
-------------------------------
 default   active   yes
 isos      active   yes
```

> ¿Qué se guarda en ellos?

Volúmenes, que serán ficheros `qcow2` o `iso` dependiendo del pool.

En el pool `default`:

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-list default
 Name                                                                 Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------
 debian-VAGRANTSLASH-bullseye64_vagrant_box_image_11.20220912.1.img   /var/lib/libvirt/images/debian-VAGRANTSLASH-bullseye64_vagrant_box_image_11.20220912.1.img
 delete.qcow2                                                         /var/lib/libvirt/images/delete.qcow2
 linux_adrian_j.qcow2                                                 /var/lib/libvirt/images/linux_adrian_j.qcow2
 prueba1.qcow2                                                        /var/lib/libvirt/images/prueba1.qcow2
 pruebas_pruebas.img                                                  /var/lib/libvirt/images/pruebas_pruebas.img
 t1-virtualizacion_pruebas.img                                        /var/lib/libvirt/images/t1-virtualizacion_pruebas.img
 vol1.qcow2                                                           /var/lib/libvirt/images/vol1.qcow2
 win10.qcow2                                                          /var/lib/libvirt/images/win10.qcow2
```

En el pool `isos`:

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-list isos
 Name                                      Path
-----------------------------------------------------------------------------------------------------
 debian-11.5.0-amd64-netinst.iso           /home/atlas/isos/debian-11.5.0-amd64-netinst.iso
 OracleLinux-R9-U0-x86_64-dvd.iso          /home/atlas/isos/OracleLinux-R9-U0-x86_64-dvd.iso
 preseed-debian-11.5.0-amd64-netinst.iso   /home/atlas/isos/preseed-debian-11.5.0-amd64-netinst.iso
 virtio-win-0.1.221.iso                    /home/atlas/isos/virtio-win-0.1.221.iso
 Win10_21H2_EnglishInternational_x64.iso   /home/atlas/isos/Win10_21H2_EnglishInternational_x64.iso
```

### Parte 2

> Mostrar información sobre el pool discos

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-info discos
Name:           discos
UUID:           f621e4c3-5e47-4da3-bfc0-7d398cec927e
State:          running
Persistent:     yes
Autostart:      no
Capacity:       27.92 GiB
Allocation:     5.98 GiB
Available:      21.94 GiB
```

### Parte 3

> Mostrar con virsh los volúmenes del pool default

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-list default
 Name                                                                 Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------
 debian-VAGRANTSLASH-bullseye64_vagrant_box_image_11.20220912.1.img   /var/lib/libvirt/images/debian-VAGRANTSLASH-bullseye64_vagrant_box_image_11.20220912.1.img
 delete.qcow2                                                         /var/lib/libvirt/images/delete.qcow2
 linux_adrian_j.qcow2                                                 /var/lib/libvirt/images/linux_adrian_j.qcow2
 prueba1.qcow2                                                        /var/lib/libvirt/images/prueba1.qcow2
 pruebas_pruebas.img                                                  /var/lib/libvirt/images/pruebas_pruebas.img
 t1-virtualizacion_pruebas.img                                        /var/lib/libvirt/images/t1-virtualizacion_pruebas.img
 vol1.qcow2                                                           /var/lib/libvirt/images/vol1.qcow2
 win10.qcow2                                                          /var/lib/libvirt/images/win10.qcow2
```

### Parte 4

> Mostrar los comandos usados para crear los dos volúmenes

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-create-as discos disco1.qcow2 --format qcow2 1G
Vol disco1.qcow2 created
```

```shell
atlas@olympus:/srv/discos$ sudo qemu-img create -f qcow2 disco2.qcow2 2G
Formatting 'disco2.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=2147483648 lazy_refcounts=off refcount_bits=16
```

> Listar los volúmenes del pool discos mostrando el aprovisionamiento ligero

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-list discos --details
 Name           Path                       Type   Capacity   Allocation
-------------------------------------------------------------------------
 disco1.qcow2   /srv/discos/disco1.qcow2   file   1.00 GiB   196.00 KiB
 disco2.qcow2   /srv/discos/disco2.qcow2   file   2.00 GiB   196.00 KiB
```

### Parte 5

> Antes de redimensionar, mostrar un `df -h`

![antesredimensionar](https://i.imgur.com/F3jsUcY.png)

### Parte 6

> Tras redimensionar, mostrar un `df -h`

![redimensioneshechas](https://i.imgur.com/XD69NZB.png)

## Desarrollo

### Ejercicio 1

> Mostrar los pools con `virsh`

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-list
 Name      State    Autostart
-------------------------------
 default   active   yes
 isos      active   yes
```

> ¿De qué tipo son?

`dir`, y lo demuestro así:

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-list --type dir
 Name      State    Autostart
-------------------------------
 default   active   yes
 isos      active   yes
```

> ¿Qué se guarda en ellos?

Volúmenes, que serán ficheros `qcow2` o `iso` dependiendo del pool.

En el pool `default`:

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-list default
 Name                                                                 Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------
 debian-VAGRANTSLASH-bullseye64_vagrant_box_image_11.20220912.1.img   /var/lib/libvirt/images/debian-VAGRANTSLASH-bullseye64_vagrant_box_image_11.20220912.1.img
 delete.qcow2                                                         /var/lib/libvirt/images/delete.qcow2
 linux_adrian_j.qcow2                                                 /var/lib/libvirt/images/linux_adrian_j.qcow2
 prueba1.qcow2                                                        /var/lib/libvirt/images/prueba1.qcow2
 pruebas_pruebas.img                                                  /var/lib/libvirt/images/pruebas_pruebas.img
 t1-virtualizacion_pruebas.img                                        /var/lib/libvirt/images/t1-virtualizacion_pruebas.img
 vol1.qcow2                                                           /var/lib/libvirt/images/vol1.qcow2
 win10.qcow2                                                          /var/lib/libvirt/images/win10.qcow2
```

En el pool `isos`:

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-list isos
 Name                                      Path
-----------------------------------------------------------------------------------------------------
 debian-11.5.0-amd64-netinst.iso           /home/atlas/isos/debian-11.5.0-amd64-netinst.iso
 OracleLinux-R9-U0-x86_64-dvd.iso          /home/atlas/isos/OracleLinux-R9-U0-x86_64-dvd.iso
 preseed-debian-11.5.0-amd64-netinst.iso   /home/atlas/isos/preseed-debian-11.5.0-amd64-netinst.iso
 virtio-win-0.1.221.iso                    /home/atlas/isos/virtio-win-0.1.221.iso
 Win10_21H2_EnglishInternational_x64.iso   /home/atlas/isos/Win10_21H2_EnglishInternational_x64.iso
```

> Mostrar los pools con `virt-manager`

![virtmanagerdefault](https://i.postimg.cc/gcB187tb/pooldefault.png)

![virtmanagerisos](https://i.postimg.cc/5932Jftq/poolisos.png)

### Ejercicio 2

> Crear un pool con virsh de características:
>
> - Nombre: discos
> - Tipo: dir
> - Directorio: /srv/discos

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-define-as discos dir --target /srv/discos
Pool discos defined
```

> Crear el directorio

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-build discos
Pool discos built
```

> Iniciar el pool

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-start discos
Pool discos started
```

> Mostrar que se ha creado el pool

```shell
atlas@olympus:~$ virsh -c qemu:///system pool-list
 Name      State    Autostart
-------------------------------
 default   active   yes
 discos    active   no
 isos      active   yes
```

### Ejercicio 3

> Mostrar los volúmenes del pool default con virsh

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-list default
 Name                                                                 Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------
 debian-VAGRANTSLASH-bullseye64_vagrant_box_image_11.20220912.1.img   /var/lib/libvirt/images/debian-VAGRANTSLASH-bullseye64_vagrant_box_image_11.20220912.1.img
 delete.qcow2                                                         /var/lib/libvirt/images/delete.qcow2
 linux_adrian_j.qcow2                                                 /var/lib/libvirt/images/linux_adrian_j.qcow2
 prueba1.qcow2                                                        /var/lib/libvirt/images/prueba1.qcow2
 pruebas_pruebas.img                                                  /var/lib/libvirt/images/pruebas_pruebas.img
 t1-virtualizacion_pruebas.img                                        /var/lib/libvirt/images/t1-virtualizacion_pruebas.img
 vol1.qcow2                                                           /var/lib/libvirt/images/vol1.qcow2
 win10.qcow2                                                          /var/lib/libvirt/images/win10.qcow2
```

> Mostrar lo mismo pero con virt-manager

![defaultvols](https://i.imgur.com/zWC0hv1.png)

### Ejercicio 4

> Crear un volumen con virsh de características:
>
> - Pool: discos
> - Nombre: disco1.qcow2
> - Tamaño: 1G

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-create-as discos disco1.qcow2 --format qcow2 1G
Vol disco1.qcow2 created
```

### Ejercicio 5

> Cambiar al directorio `/srv/discos`

```shell
atlas@olympus:~$ cd /srv/discos
atlas@olympus:/srv/discos$
```

> Crear una imagen con `qemu-img` de características:
>
> - Nombre: disco2.qcow2
> - Tamaño: 2G

```shell
atlas@olympus:/srv/discos$ sudo qemu-img create -f qcow2 disco2.qcow2 2G
Formatting 'disco2.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=2147483648 lazy_refcounts=off refcount_bits=16
```

> Refrescar el pool para que se reconozca como volumen

```shell
atlas@olympus:/srv/discos$ virsh -c qemu:///system pool-refresh discos
Pool discos refreshed
```

> Mostrar los volúmenes del pool discos para comprobar que todo es correcto

```shell
atlas@olympus:/srv/discos$ virsh -c qemu:///system vol-list discos
 Name           Path
------------------------------------------
 disco1.qcow2   /srv/discos/disco1.qcow2
 disco2.qcow2   /srv/discos/disco2.qcow2
```

### Ejercicio 6

> ¿Qué característica especial tienen los qcow2?

Aprovisionamiento ligero, es decir, el espacio real ocupado en disco irá creciendo progresivamente.

La VM verá un tamaño virtual del fichero, que será el máximo que éste podrá alcanzar.

> Listar los volúmenes del pool discos mostrando el aprovisionamiento ligero

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-list discos --details
 Name           Path                       Type   Capacity   Allocation
-------------------------------------------------------------------------
 disco1.qcow2   /srv/discos/disco1.qcow2   file   1.00 GiB   196.00 KiB
 disco2.qcow2   /srv/discos/disco2.qcow2   file   2.00 GiB   196.00 KiB
```

### Ejercicio 7

> Añadir `disco1.qcow2` a `debian_adrian_j` con virsh

```shell
atlas@olympus:~$ virsh -c qemu:///system attach-disk debian_adrian_j /srv/discos/disco1.qcow2 vdb --driver=qemu --type disk --subdriver qcow2 --persistent
Disk attached successfully
```

> Añadir `disco2.qcow2` con virt-manager

![disco2virtmanager](https://i.imgur.com/FDSeTSL.png)

> Mostrar que están los 2 discos añadidos:

![disco12añadidos](https://i.imgur.com/0qDKLFW.png)

> Formatear ambos discos

![disco12formateo](https://i.imgur.com/1WYmt9x.png)

> Montarlos persistentemente

Obtener los UUID de ambos discos:

![disco12UUIDs](https://i.imgur.com/hDfgf8e.png)

Añadir las siguientes líneas al final de `/etc/fstab`:

![disco12persistent](https://i.imgur.com/hFBpsIc.png)

Hacer los cambios efectivos:

![disco12montados](https://i.imgur.com/sqDIIx9.png)

Al reiniciar, veríamos que los montajes se mantienen

### Ejercicio 8

> Parar `debian_adrian_j`

```shell
atlas@olympus:~$ virsh -c qemu:///system destroy debian_adrian_j
Domain 'debian_adrian_j' destroyed
```

> Redimensionar con virsh `disco1.qcow2` a 2G

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-resize disco1.qcow2 2G --pool discos
Size of volume 'disco1.qcow2' successfully changed to 2G
```

> Redimensionar con qemu-img `disco2.qcow2` a 3G

```shell
atlas@olympus:~$ sudo qemu-img resize /srv/discos/disco2.qcow2 3G
Image resized.
```

> Mostrar que los discos se han redimensionado correctamente pero los sistemas de ficheros no

![filesystemsinredimensionar](https://i.imgur.com/aJiJaoT.png)

> Desmontar, redimensionar los sistemas de ficheros de ambos discos, y volver a montar

![redimensiones](https://i.imgur.com/7gj7ND9.png)

> Mostrar que los sistemas de ficheros se han redimensionado correctamente

![redimensioneshechas](https://i.imgur.com/XD69NZB.png)
