---
title: "Ejercicios de manejo de módulos"
---

# Ejercicio 1

## Comprueba los módulos cargados en tu equipo

`lsmod`

# Ejercicio 2

## Cuenta el número de módulos disponibles en el núcleo que estás usando

find /lib/modules/$(uname -r)/kernel -type f -iname *.ko | wc -l


# Ejercicio 3

## Conecta un lápiz USB y observa la salida de la instrucción sudo dmesg

```
[ 4593.172666] blk_update_request: I/O error, dev sdb, sector 2049 op 0x1:(WRITE) flags 0x0 phys_seg 1 prio class 0
[ 4593.172678] Buffer I/O error on dev sdb1, logical block 1, lost async page write
[ 4593.200317] FAT-fs (sdb1): unable to read boot sector to mark fs as dirty
[ 4593.482408] usb 1-2: new high-speed USB device number 6 using xhci_hcd
[ 4647.106080] usb 1-2: new high-speed USB device number 7 using xhci_hcd
[ 4647.254940] usb 1-2: New USB device found, idVendor=abcd, idProduct=1234, bcdDevice= 1.00
[ 4647.254947] usb 1-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 4647.254951] usb 1-2: Product: UDisk           
[ 4647.254954] usb 1-2: Manufacturer: General 
[ 4647.254957] usb 1-2: SerialNumber: Љ
[ 4647.257334] usb-storage 1-2:1.0: USB Mass Storage device detected
[ 4647.257962] scsi host5: usb-storage 1-2:1.0
[ 4648.278641] scsi 5:0:0:0: Direct-Access     General  UDisk            5.00 PQ: 0 ANSI: 2
[ 4648.279187] sd 5:0:0:0: Attached scsi generic sg1 type 0
[ 4648.279543] sd 5:0:0:0: [sdb] 7866368 512-byte logical blocks: (4.03 GB/3.75 GiB)
[ 4648.279749] sd 5:0:0:0: [sdb] Write Protect is off
[ 4648.279754] sd 5:0:0:0: [sdb] Mode Sense: 0b 00 00 08
[ 4648.279901] sd 5:0:0:0: [sdb] No Caching mode page found
[ 4648.279905] sd 5:0:0:0: [sdb] Assuming drive cache: write through
[ 4648.300684]  sdb: sdb1
[ 4648.302644] sd 5:0:0:0: [sdb] Attached SCSI removable disk
[ 4648.729316] FAT-fs (sdb1): Volume was not properly unmounted. Some data may be corrupt. Please run fsck.
[ 4704.810503] usb 1-2: USB disconnect, device number 7
[ 4722.273558] usb 1-3: new high-speed USB device number 8 using xhci_hcd
[ 4722.430429] usb 1-3: New USB device found, idVendor=abcd, idProduct=1234, bcdDevice= 1.00
[ 4722.430437] usb 1-3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 4722.430441] usb 1-3: Product: UDisk           
[ 4722.430445] usb 1-3: Manufacturer: General 
[ 4722.430448] usb 1-3: SerialNumber: Љ
[ 4722.432116] usb-storage 1-3:1.0: USB Mass Storage device detected
[ 4722.432500] scsi host5: usb-storage 1-3:1.0
[ 4723.446056] scsi 5:0:0:0: Direct-Access     General  UDisk            5.00 PQ: 0 ANSI: 2
[ 4723.446893] sd 5:0:0:0: Attached scsi generic sg1 type 0
[ 4723.447188] sd 5:0:0:0: [sdb] 7866368 512-byte logical blocks: (4.03 GB/3.75 GiB)
[ 4723.447389] sd 5:0:0:0: [sdb] Write Protect is off
[ 4723.447398] sd 5:0:0:0: [sdb] Mode Sense: 0b 00 00 08
[ 4723.447549] sd 5:0:0:0: [sdb] No Caching mode page found
[ 4723.447553] sd 5:0:0:0: [sdb] Assuming drive cache: write through
[ 4723.484232]  sdb: sdb1
[ 4723.485775] sd 5:0:0:0: [sdb] Attached SCSI removable disk
[ 4723.900890] FAT-fs (sdb1): Volume was not properly unmounted. Some data may be corrupt. Please run fsck.
```

# Ejercicio 4

## Elimina el módulo correspondiente a algún dispotivo no esencial y comprueba qué ocurre. Vuelve a cargarlo.

`sudo modprobe -r psmouse`

`sudo modprobe psmouse`

`lsmod | grep psmouse`


# Ejercicio 5

## Selecciona un módulo que esté en uso en tu equipo y configura el arranque para que no se cargue automáticamente

Create a file '/etc/modprobe.d/<modulename>.conf' containing 'blacklist <modulename>'.

Un fichero de configuración por cada módulo, y dentro de la configuración del módulo, blacklist o no. Hay otros parámetros además como options, install...etc.

# Ejercicio 6

## Carga el módulo loop, obtén información de qué es y para qué sirve. Lista el contenido de /sys/modules/loop/parameters y configura el equipo para que se puedan cargar hasta 32 dispositvos loop la próxima vez que se arranque













