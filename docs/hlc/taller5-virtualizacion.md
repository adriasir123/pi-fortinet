# Taller 5: Clonación e instantáneas de máquinas virtuales

La teoría para completar este taller se encuentra en el [capítulo 6 del curso de virtualización](https://github.com/josedom24/curso_virtualizacion_linux).

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

### Parte 2

> Listar las VMs para que se vea `plantilla-linux`

```shell
atlas@olympus:~$ virsh -c qemu:///system list --all
 Id   Name                        State
--------------------------------------------
 -    debian11                    shut off
 -    debian_adrian_j             shut off
 -    plantilla-linux             shut off
 -    pruebas_pruebas             shut off
 -    t1-virtualizacion_pruebas   shut off
 -    win10                       shut off
```

> Captura donde se vea que da un error al iniciar `plantilla-linux`

![errorplantilla](https://i.imgur.com/gVj5oIY.png)

### Parte 3

> Captura de `clone-full` mostrando su IP

![ipclonefull](https://i.imgur.com/c7ZwUO8.png)

> Captura accediendo por SSH

![sshclonefull](https://i.imgur.com/rUZuU28.png)

### Parte 4

> Listar las VMs para que se vea `clone-link`

```shell
atlas@olympus:~$ virsh -c qemu:///system list --all
 Id   Name                        State
--------------------------------------------
 -    clone-full                  shut off
 -    clone-link                  shut off
 -    debian11                    shut off
 -    debian_adrian_j             shut off
 -    plantilla-linux             shut off
 -    pruebas_pruebas             shut off
 -    t1-virtualizacion_pruebas   shut off
 -    win10                       shut off
```

> Mostrar información sobre el volumen `clone-link.qcow2`, para comprobar que tiene un backing store

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-info clone-link.qcow2 default
Name:           clone-link.qcow2
Type:           file
Capacity:       20.00 GiB
Allocation:     196.00 KiB
```

De forma directa con la salida de este comando no podemos saber si tiene un backing store, aunque lo podemos intuir por su mínimo espacio real en disco, pero si mostramos su XML directamente se ve claro:

![clonelinkxml](https://i.imgur.com/uiPDrpw.png)

> ¿Por qué ocupa tan poco en disco?

Si comparamos el espacio real ocupado por `clone-full` y `clone-link` vemos que hay una clara diferencia:

![diferenciaespacio](https://i.imgur.com/f6lrRJv.png)

La razón es que el volumen `clone-link.qcow2` está usando como imagen base (backing store) el volumen de la VM `plantilla-linux`.  
En `clone-link.qcow2` sólo se irán guardando los cambios.

### Parte 5

> Crear un directorio en `clone-full`

![directorio](https://i.imgur.com/3P0IUB2.png)

> Hacer una snapshot

![snapshot1](https://i.imgur.com/W51L4yJ.png)

> Borrar el directorio

![rmdirectorio](https://i.imgur.com/54V4n2l.png)

> Revertir el estado usando la snapshot

![snapshotrevertida](https://i.imgur.com/eNcvfwS.png)

> Comprobar que ha funcionado

![maquinarevertida](https://i.imgur.com/J7eTlhN.png)

> Mostrar la lista de snapshots de clone-full

```shell
atlas@olympus:~$ virsh -c qemu:///system snapshot-list clone-full
 Name        Creation Time               State
--------------------------------------------------
 snapshot1   2022-10-15 02:22:26 +0200   running
```

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

### Ejercicio 2

> Crear una plantilla a partir de `maquina-clonada` con nombre `plantilla-linux`

Teniendo `maquina-clonada` apagada, hago la generalización:

```shell
atlas@olympus:~$ sudo virt-sysprep -d maquina-clonada --hostname plantilla-linux
[sudo] password for atlas:
[   0.0] Examining the guest ...
[   7.3] Performing "abrt-data" ...
[   7.3] Performing "backup-files" ...
[   7.5] Performing "bash-history" ...
[   7.5] Performing "blkid-tab" ...
[   7.5] Performing "crash-data" ...
[   7.6] Performing "cron-spool" ...
[   7.6] Performing "dhcp-client-state" ...
[   7.6] Performing "dhcp-server-state" ...
[   7.6] Performing "dovecot-data" ...
[   7.6] Performing "ipa-client" ...
[   7.6] Performing "kerberos-hostkeytab" ...
[   7.6] Performing "logfiles" ...
[   7.8] Performing "machine-id" ...
[   7.8] Performing "mail-spool" ...
[   7.8] Performing "net-hostname" ...
[   7.8] Performing "net-hwaddr" ...
[   7.9] Performing "pacct-log" ...
[   7.9] Performing "package-manager-cache" ...
[   7.9] Performing "pam-data" ...
[   8.0] Performing "passwd-backups" ...
[   8.0] Performing "puppet-data-log" ...
[   8.0] Performing "rh-subscription-manager" ...
[   8.0] Performing "rhn-systemid" ...
[   8.1] Performing "rpm-db" ...
[   8.1] Performing "samba-db-log" ...
[   8.1] Performing "script" ...
[   8.1] Performing "smolt-uuid" ...
[   8.2] Performing "ssh-hostkeys" ...
[   8.2] Performing "ssh-userdir" ...
[   8.2] Performing "sssd-db-log" ...
[   8.2] Performing "tmp-files" ...
[   8.3] Performing "udev-persistent-net" ...
[   8.3] Performing "utmp" ...
[   8.3] Performing "yum-uuid" ...
[   8.3] Performing "customize" ...
[   8.4] Setting a random seed
[   8.4] Setting the machine ID in /etc/machine-id
[   8.4] Setting the hostname: plantilla-linux
[   9.2] Performing "lvm-uuids" ...
```

Cambio también su nombre con virsh para que sea explicativo:

```shell
atlas@olympus:~$ virsh -c qemu:///system domrename maquina-clonada plantilla-linux
Domain successfully renamed
```

Pongo su disco en modo lectura:

```shell
sudo chmod -w /var/lib/libvirt/images/maquina-clonada.qcow2
```

Da un error si intentamos iniciarla:

![errorplantilla](https://i.imgur.com/gVj5oIY.png)

### Ejercicio 3

> Con `virt-manager` hacer una clonación completa de `plantilla-linux` de nombre `clone-full`

![clonefull](https://i.imgur.com/udsijwT.png)

> Acceder por ssh a `clone-full`

Averiguo su IP:

```shell
atlas@olympus:~$ virsh -c qemu:///system domifaddr clone-full
 Name       MAC address          Protocol     Address
-------------------------------------------------------------------------------
 vnet3      52:54:00:ed:7a:2a    ipv4         192.168.122.68/24
```

Muestro que falla el acceso:

```shell
atlas@olympus:~$ ssh debian@192.168.122.68
ssh: connect to host 192.168.122.68 port 22: Connection refused
```

Regenero las claves SSH de host en `clone-full`, y de paso le cambio el hostname:

![regenerohostssh](https://i.imgur.com/fAc2vEy.png)

Tras un reinicio, ya podríamos acceder:

```shell
atlas@olympus:~$ ssh debian@192.168.122.68
The authenticity of host '192.168.122.68 (192.168.122.68)' can't be established.
ECDSA key fingerprint is SHA256:W8Hf46WFG8uHS2ToxowuTgoVom7eLMVABL6BG20FZyQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.122.68' (ECDSA) to the list of known hosts.
debian@192.168.122.68's password:
Linux clone-full 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Oct 14 12:58:52 2022
debian@clone-full:~$
```

### Ejercicio 4

> Hacer una clonación enlazada de `plantilla-linux` con nombre `clone-link`

Comprobar el tamaño del volumen del que partiremos:

```shell
atlas@olympus:~$ virsh -c qemu:///system domblkinfo plantilla-linux vda --human
Capacity:       20.000 GiB
Allocation:     1.812 GiB
Physical:       1.811 GiB
```

Creo el nuevo volumen usando como backing store el de `plantilla-linux`:

```shell
atlas@olympus:~$ virsh -c qemu:///system vol-create-as default clone-link.qcow2 20G --format qcow2 --backing-vol maquina-clonada.qcow2 --backing-vol-format qcow2
Vol clone-link.qcow2 created
```

Hago la clonación enlazada:

```shell
atlas@olympus:~$ virt-clone --connect=qemu:///system --original plantilla-linux --name clone-link --file /var/lib/libvirt/images/clone-link.qcow2 --preserve-data

Clone 'clone-link' created successfully.
```

### Ejercicio 5

> Crear un directorio en `clone-full`

![directorio](https://i.imgur.com/3P0IUB2.png)

> Hacer una snapshot

```shell
atlas@olympus:~$ virsh -c qemu:///system snapshot-create-as clone-full --name snapshot1 --description "Critical directory created" --atomic
Domain snapshot snapshot1 created
```

> Borrar el directorio

![rmdirectorio](https://i.imgur.com/54V4n2l.png)

> Revertir el estado usando la snapshot

```shell
virsh -c qemu:///system snapshot-revert clone-full snapshot1
```

> Comprobar que ha funcionado

![maquinarevertida](https://i.imgur.com/J7eTlhN.png)
