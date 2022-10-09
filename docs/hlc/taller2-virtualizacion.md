# Taller 2: Creación de máquinas virtuales desde la línea de comandos

La teoría para completar este taller se encuentra en el [capítulo 3 del curso de virtualización](https://github.com/josedom24/curso_virtualizacion_linux).

## Entrega

### Parte 1

> Entregar el comando usado para crear la VM

```shell
virt-install --connect qemu:///system \
			 --virt-type kvm \
			 --name linux_adrian_j \
			 --cdrom ~/isos/debian-11.5.0-amd64-netinst.iso \
			 --os-variant debian10 \
			 --disk size=15 \
			 --memory 2048 \
			 --vcpus 2
```

Si no queremos GUI, dejamos este paso de la instalación de la siguiente manera:

![singui](https://i.imgur.com/Ory3CKR.png)

### Parte 2

> Entregar una captura del acceso a la VM con virt-viewer

```shell
virt-viewer -c qemu:///system linux_adrian_j
```

![acceso-virt-viewer](https://i.imgur.com/bA8IExy.png)

### Parte 3

> Entregar la lista de VMs creadas

```shell
atlas@olympus:~$ virsh -c qemu:///system list --all
 Id   Name                        State
--------------------------------------------
 3    linux_adrian_j              running
 -    prueba2                     shut off
 -    pruebas_pruebas             shut off
 -    t1-virtualizacion_pruebas   shut off
```

### Parte 4

> Mostrar el volumen creado en el pool default

![volumencreado](https://i.imgur.com/c1iWHSH.png)

> Mostrar que en `/var/lib/libvirt/images` se ha creado un qcow2

![ficheroqcow2](https://i.imgur.com/baF8kxT.png)

### Parte 5

> Mostrar la información de la VM

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo linux_adrian_j
Id:             -
Name:           linux_adrian_j
UUID:           29cc9a23-f712-4649-a61f-eea1cb11599e
OS Type:        hvm
State:          shut off
CPU(s):         2
Max memory:     2097152 KiB
Used memory:    2097152 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: apparmor
Security DOI:   0
```

> Mostrar la IP

```shell
atlas@olympus:~$ virsh -c qemu:///system domifaddr linux_adrian_j
 Name       MAC address          Protocol     Address
-------------------------------------------------------------------------------
 vnet5      52:54:00:42:b7:c6    ipv4         192.168.122.47/24
```

> Mostrar los discos

```shell
atlas@olympus:~$ virsh -c qemu:///system domblklist linux_adrian_j
 Target   Source
--------------------------------------------------------
 vda      /var/lib/libvirt/images/linux_adrian_j.qcow2
 sda      -
```

### Parte 6

#### Cambio de nombre

Apago la máquina:

```shell
atlas@olympus:~$ virsh -c qemu:///system destroy linux_adrian_j
Domain 'linux_adrian_j' destroyed
```

Cambio el nombre:

```shell
atlas@olympus:~$ virsh -c qemu:///system domrename linux_adrian_j debian_adrian_j
Domain successfully renamed
```

Muestro que se ha aplicado:

```shell
atlas@olympus:~$ virsh -c qemu:///system list --all
 Id   Name                        State
--------------------------------------------
 -    debian_adrian_j             shut off
 -    prueba2                     shut off
 -    pruebas_pruebas             shut off
 -    t1-virtualizacion_pruebas   shut off
```

#### Cambio de CPUs

Muestro la cantidad actual:

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo debian_adrian_j | grep CPU
CPU(s):         2
```

Edito el XML para que cambie el valor:

```shell
virsh -c qemu:///system edit debian_adrian_j
```

![cambiocpu](https://i.imgur.com/KG0Y7yB.png)

Muestro que se ha aplicado:

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo debian_adrian_j | grep CPU
CPU(s):         1
```

#### Cambio de RAM

Muestro la cantidad actual:

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo debian_adrian_j | grep memory
Max memory:     2097152 KiB
Used memory:    2097152 KiB
```

Edito el XML para que cambie el valor:

```shell
virsh -c qemu:///system edit debian_adrian_j
```

![cambioram](https://i.imgur.com/qfZtil0.png)

Muestro que se ha aplicado:

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo debian_adrian_j | grep memory
Max memory:     1048576 KiB
Used memory:    1048576 KiB
```

## Desarrollo

### Ejercicio 1

> Crear una VM con características:
>
> - Nombre: linux_adrian_j
> - Disco: 15 GB
> - RAM: 2 GB
> - CPUs: 2
> - Sin GUI

```shell
virt-install --connect qemu:///system \
			 --virt-type kvm \
			 --name linux_adrian_j \
			 --cdrom ~/isos/debian-11.5.0-amd64-netinst.iso \
			 --os-variant debian10 \
			 --disk size=15 \
			 --memory 2048 \
			 --vcpus 2
```

Si no queremos GUI, dejamos este paso de la instalación de la siguiente manera:

![singui](https://i.imgur.com/Ory3CKR.png)

### Ejercicio 2

> Acceder usando virt-viewer

```shell
virt-viewer -c qemu:///system linux_adrian_j
```

![acceso-virt-viewer](https://i.imgur.com/bA8IExy.png)

### Ejercicio 3

> Comprobar la IP, disco y RAM

![comprobaciones](https://i.imgur.com/o1KMrk5.png)

### Ejercicio 4

> Listar las máquinas

```shell
atlas@olympus:~$ virsh -c qemu:///system list --all
 Id   Name                        State
--------------------------------------------
 3    linux_adrian_j              running
 -    prueba2                     shut off
 -    pruebas_pruebas             shut off
 -    t1-virtualizacion_pruebas   shut off
```

> Apagar `linux_adrian_j`

```shell
atlas@olympus:~$ virsh -c qemu:///system shutdown linux_adrian_j
Domain 'linux_adrian_j' is being shutdown
```

> Volver a encenderla

```shell
atlas@olympus:~$ virsh -c qemu:///system start linux_adrian_j
Domain 'linux_adrian_j' started
```

> Reiniciarla

```shell
atlas@olympus:~$ virsh -c qemu:///system reboot linux_adrian_j
Domain 'linux_adrian_j' is being rebooted
```

### Ejercicio 5

> Mostrar el XML de `linux_adrian_j`

```shell
atlas@olympus:~$ virsh -c qemu:///system dumpxml linux_adrian_j
<domain type='kvm' id='4'>
  <name>linux_adrian_j</name>
  <uuid>29cc9a23-f712-4649-a61f-eea1cb11599e</uuid>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://debian.org/debian/10"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory unit='KiB'>2097152</memory>
  <currentMemory unit='KiB'>2097152</currentMemory>
  <vcpu placement='static'>2</vcpu>
  <resource>
    <partition>/machine</partition>
  </resource>
  <os>
    <type arch='x86_64' machine='pc-q35-5.2'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <vmport state='off'/>
  </features>
  <cpu mode='custom' match='exact' check='full'>
    <model fallback='forbid'>Skylake-Client-IBRS</model>
    <vendor>Intel</vendor>
    <feature policy='require' name='ss'/>
    <feature policy='require' name='vmx'/>
    <feature policy='require' name='pdcm'/>
    <feature policy='require' name='hypervisor'/>
    <feature policy='require' name='tsc_adjust'/>
    <feature policy='require' name='clflushopt'/>
    <feature policy='require' name='umip'/>
    <feature policy='require' name='stibp'/>
    <feature policy='require' name='arch-capabilities'/>
    <feature policy='require' name='ssbd'/>
    <feature policy='require' name='xsaves'/>
    <feature policy='require' name='pdpe1gb'/>
    <feature policy='require' name='ibpb'/>
    <feature policy='require' name='amd-stibp'/>
    <feature policy='require' name='amd-ssbd'/>
    <feature policy='require' name='skip-l1dfl-vmentry'/>
    <feature policy='require' name='pschange-mc-no'/>
    <feature policy='disable' name='hle'/>
    <feature policy='disable' name='rtm'/>
    <feature policy='disable' name='mpx'/>
  </cpu>
  <clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/var/lib/libvirt/images/linux_adrian_j.qcow2' index='2'/>
      <backingStore/>
      <target dev='vda' bus='virtio'/>
      <alias name='virtio-disk0'/>
      <address type='pci' domain='0x0000' bus='0x04' slot='0x00' function='0x0'/>
    </disk>
    <disk type='file' device='cdrom'>
      <driver name='qemu'/>
      <target dev='sda' bus='sata'/>
      <readonly/>
      <alias name='sata0-0-0'/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
    <controller type='usb' index='0' model='qemu-xhci' ports='15'>
      <alias name='usb'/>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x00' function='0x0'/>
    </controller>
    <controller type='sata' index='0'>
      <alias name='ide'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x1f' function='0x2'/>
    </controller>
    <controller type='pci' index='0' model='pcie-root'>
      <alias name='pcie.0'/>
    </controller>
    <controller type='pci' index='1' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='1' port='0x10'/>
      <alias name='pci.1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0' multifunction='on'/>
    </controller>
    <controller type='pci' index='2' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='2' port='0x11'/>
      <alias name='pci.2'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x1'/>
    </controller>
    <controller type='pci' index='3' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='3' port='0x12'/>
      <alias name='pci.3'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x2'/>
    </controller>
    <controller type='pci' index='4' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='4' port='0x13'/>
      <alias name='pci.4'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x3'/>
    </controller>
    <controller type='pci' index='5' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='5' port='0x14'/>
      <alias name='pci.5'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x4'/>
    </controller>
    <controller type='pci' index='6' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='6' port='0x15'/>
      <alias name='pci.6'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x5'/>
    </controller>
    <controller type='pci' index='7' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='7' port='0x16'/>
      <alias name='pci.7'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x6'/>
    </controller>
    <controller type='virtio-serial' index='0'>
      <alias name='virtio-serial0'/>
      <address type='pci' domain='0x0000' bus='0x03' slot='0x00' function='0x0'/>
    </controller>
    <interface type='network'>
      <mac address='52:54:00:42:b7:c6'/>
      <source network='default' portid='4abfa2b2-cfc7-4d67-9afa-134f82f939b0' bridge='virbr0'/>
      <target dev='vnet3'/>
      <model type='virtio'/>
      <alias name='net0'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </interface>
    <serial type='pty'>
      <source path='/dev/pts/1'/>
      <target type='isa-serial' port='0'>
        <model name='isa-serial'/>
      </target>
      <alias name='serial0'/>
    </serial>
    <console type='pty' tty='/dev/pts/1'>
      <source path='/dev/pts/1'/>
      <target type='serial' port='0'/>
      <alias name='serial0'/>
    </console>
    <channel type='unix'>
      <source mode='bind' path='/var/lib/libvirt/qemu/channel/target/domain-4-linux_adrian_j/org.qemu.guest_agent.0'/>
      <target type='virtio' name='org.qemu.guest_agent.0' state='connected'/>
      <alias name='channel0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0' state='disconnected'/>
      <alias name='channel1'/>
      <address type='virtio-serial' controller='0' bus='0' port='2'/>
    </channel>
    <input type='tablet' bus='usb'>
      <alias name='input0'/>
      <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'>
      <alias name='input1'/>
    </input>
    <input type='keyboard' bus='ps2'>
      <alias name='input2'/>
    </input>
    <graphics type='spice' port='5900' autoport='yes' listen='127.0.0.1'>
      <listen type='address' address='127.0.0.1'/>
      <image compression='off'/>
    </graphics>
    <sound model='ich9'>
      <alias name='sound0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x1b' function='0x0'/>
    </sound>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <alias name='video0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
    </video>
    <redirdev bus='usb' type='spicevmc'>
      <alias name='redir0'/>
      <address type='usb' bus='0' port='2'/>
    </redirdev>
    <redirdev bus='usb' type='spicevmc'>
      <alias name='redir1'/>
      <address type='usb' bus='0' port='3'/>
    </redirdev>
    <memballoon model='virtio'>
      <alias name='balloon0'/>
      <address type='pci' domain='0x0000' bus='0x05' slot='0x00' function='0x0'/>
    </memballoon>
    <rng model='virtio'>
      <backend model='random'>/dev/urandom</backend>
      <alias name='rng0'/>
      <address type='pci' domain='0x0000' bus='0x06' slot='0x00' function='0x0'/>
    </rng>
  </devices>
  <seclabel type='dynamic' model='apparmor' relabel='yes'>
    <label>libvirt-29cc9a23-f712-4649-a61f-eea1cb11599e</label>
    <imagelabel>libvirt-29cc9a23-f712-4649-a61f-eea1cb11599e</imagelabel>
  </seclabel>
  <seclabel type='dynamic' model='dac' relabel='yes'>
    <label>+64055:+64055</label>
    <imagelabel>+64055:+64055</imagelabel>
  </seclabel>
</domain>
```

### Ejercicio 6

#### 6.1

> Modificar el nombre de la máquina `linux_adrian_j`

Apago la máquina:

```shell
atlas@olympus:~$ virsh -c qemu:///system destroy linux_adrian_j
Domain 'linux_adrian_j' destroyed
```

Cambio el nombre:

```shell
atlas@olympus:~$ virsh -c qemu:///system domrename linux_adrian_j debian_adrian_j
Domain successfully renamed
```

Muestro que se ha aplicado:

```shell
atlas@olympus:~$ virsh -c qemu:///system list --all
 Id   Name                        State
--------------------------------------------
 -    debian_adrian_j             shut off
 -    prueba2                     shut off
 -    pruebas_pruebas             shut off
 -    t1-virtualizacion_pruebas   shut off
```

#### 6.2

> Modificar el número de CPUs

Muestro la cantidad actual:

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo debian_adrian_j | grep CPU
CPU(s):         2
```

Edito el XML para que cambie el valor:

```shell
virsh -c qemu:///system edit debian_adrian_j
```

![cambiocpu](https://i.imgur.com/KG0Y7yB.png)

Muestro que se ha aplicado:

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo debian_adrian_j | grep CPU
CPU(s):         1
```

#### 6.3

> Modificar la RAM

Muestro la cantidad actual:

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo debian_adrian_j | grep memory
Max memory:     2097152 KiB
Used memory:    2097152 KiB
```

Edito el XML para que cambie el valor:

```shell
virsh -c qemu:///system edit debian_adrian_j
```

![cambioram](https://i.imgur.com/qfZtil0.png)

Muestro que se ha aplicado:

```shell
atlas@olympus:~$ virsh -c qemu:///system dominfo debian_adrian_j | grep memory
Max memory:     1048576 KiB
Used memory:    1048576 KiB
```
