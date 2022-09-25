# Instalación de Debian 11

## Problemas durante la instalación

- **No detectaba el SSD**: durante el paso de particionamiento, sólo se reconocía el HDD. Se soluciona cambiando el modo SATA de RST a AHCI en la BIOS.
- **Error `Unable to install GRUB in dummy`**: sucede durante el paso de instalación de GRUB, y significa que el instalador de debian no puede acceder a EFI para instalar GRUB. El error ocurre cuando la partición EFI está dentro de LVM, ya que el instalador de debian parece no ser capaz de acceder a LVM para instalar GRUB. Se soluciona sacando la partición EFI como partición física fuera de LVM.

## Esquema de particionamiento

Tengo los siguientes discos físicos, un SSD y un HDD:

```bash
atlas@olympus:~$ sudo hwinfo --disk --short
disk:
  /dev/nvme0n1         Kingston Technology Company Disk
  /dev/sda             ST500LT012-1DG14
```

El particionamiento está hecho con LVM, a excepción de la partición EFI, que es una partición física fuera de LVM por la razón ya explicada anteriormente.

**¿Por qué utilizamos LVM en lugar de particiones físicas?**  
En LVM asignamos "pools" de espacio y a partir de ahí creamos volúmenes, y éstos son totalmente dinámicos. Podemos redimensionar, crear y borrar como queramos y cuando queramos sin problemas.  
Esto evidentemente es una gran ventaja en comparación con particiones físicas tradicionales, que no ofrecen demasiada flexibilidad sobre todo a la hora de redimensionar.

Muestro mis volúmenes físicos de LVM:

```bash
atlas@olympus:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/nvme0n1p2
  VG Name               SSD
  PV Size               238.38 GiB / not usable 3.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              61025
  Free PE               11197
  Allocated PE          49828
  PV UUID               RXPn3u-nkYK-uiN5-Qncv-AnAf-qZaB-M6ki78

  --- Physical volume ---
  PV Name               /dev/sda1
  VG Name               HDD
  PV Size               <465.76 GiB / not usable 2.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              119234
  Free PE               47709
  Allocated PE          71525
  PV UUID               K0vDwq-sWWO-9BGf-qniy-pdjj-JmH3-73rk5a
```

Muestro mis grupos de volúmenes de LVM:

```bash
atlas@olympus:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               SSD
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                4
  Open LV               4
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <238.38 GiB
  PE Size               4.00 MiB
  Total PE              61025
  Alloc PE / Size       49828 / 194.64 GiB
  Free  PE / Size       11197 / <43.74 GiB
  VG UUID               Gd9Ehq-9T2H-N62g-5jl6-p4tt-KEIJ-53olaH

  --- Volume group ---
  VG Name               HDD
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <465.76 GiB
  PE Size               4.00 MiB
  Total PE              119234
  Alloc PE / Size       71525 / 279.39 GiB
  Free  PE / Size       47709 / 186.36 GiB
  VG UUID               vQpT3a-9uE7-ZL4x-Dlfq-uQkg-KDRd-hdjggm
```

Si nos damos cuenta, tengo 2 grupos de volúmenes, SSD y HDD.  
Mi disco físico SSD está en el grupo SSD y mi disco físico HDD está en el grupo HDD.

Aunque tenga sólo 1 volumen físico por grupo de volúmenes, es muy importante hacer esta división para mí. Así tengo un control real sobre qué volúmenes lógicos necesitan velocidad, y por tanto van en el grupo SSD, y qué volúmenes lógicos no necesitan velocidad, y por tanto van en el grupo HDD.

Muestro mis volúmenes lógicos de LVM:

```bash
atlas@olympus:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/SSD/root
  LV Name                root
  VG Name                SSD
  LV UUID                1nsB1N-F7yF-p22v-7qXX-Cb9U-uQk0-g8kf7G
  LV Write Access        read/write
  LV Creation host, time olympus, 2022-09-18 18:10:38 +0200
  LV Status              available
  # open                 1
  LV Size                <27.94 GiB
  Current LE             7152
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:0

  --- Logical volume ---
  LV Path                /dev/SSD/boot
  LV Name                boot
  VG Name                SSD
  LV UUID                LMdf6Z-Uc8s-B1Xf-dVVv-Ges0-VIVx-nBqv0X
  LV Write Access        read/write
  LV Creation host, time olympus, 2022-09-18 18:12:50 +0200
  LV Status              available
  # open                 1
  LV Size                952.00 MiB
  Current LE             238
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:1

  --- Logical volume ---
  LV Path                /dev/SSD/swap
  LV Name                swap
  VG Name                SSD
  LV UUID                PyIo9Y-UjDh-WGES-xPAO-SDuA-fpqm-P2GEVN
  LV Write Access        read/write
  LV Creation host, time olympus, 2022-09-18 18:13:23 +0200
  LV Status              available
  # open                 2
  LV Size                <7.45 GiB
  Current LE             1907
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:2

  --- Logical volume ---
  LV Path                /dev/SSD/var
  LV Name                var
  VG Name                SSD
  LV UUID                AYy3Cv-GfJY-929d-bhhz-dugm-DWrb-UL2Qod
  LV Write Access        read/write
  LV Creation host, time olympus, 2022-09-18 18:13:36 +0200
  LV Status              available
  # open                 1
  LV Size                158.32 GiB
  Current LE             40531
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:3

  --- Logical volume ---
  LV Path                /dev/HDD/home
  LV Name                home
  VG Name                HDD
  LV UUID                BdQutw-r9An-lHVS-mntV-j5WM-cyBY-xqufHY
  LV Write Access        read/write
  LV Creation host, time olympus, 2022-09-18 17:55:42 +0200
  LV Status              available
  # open                 1
  LV Size                279.39 GiB
  Current LE             71525
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:4
```

Una vez mostrada toda la información relevante sobre LVM, puedo mostrar la situación general resumida del particionado para que todo se entienda mejor de un vistazo:

```bash
atlas@olympus:~$ lsblk
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda            8:0    0 465.8G  0 disk
└─sda1         8:1    0 465.8G  0 part
  └─HDD-home 254:4    0 279.4G  0 lvm  /home
nvme0n1      259:0    0 238.5G  0 disk
├─nvme0n1p1  259:1    0    94M  0 part /boot/efi
└─nvme0n1p2  259:2    0 238.4G  0 part
  ├─SSD-root 254:0    0  27.9G  0 lvm  /
  ├─SSD-boot 254:1    0   952M  0 lvm  /boot
  ├─SSD-swap 254:2    0   7.4G  0 lvm  [SWAP]
  └─SSD-var  254:3    0 158.3G  0 lvm  /var
```

Razonamientos sobre particiones y tamaños:

- El LV Home es lo único que se encuentra en el HDD porque es el único volúmen que no necesita velocidad de los que tengo. Le asigné 300 GB, aunque podría haberle asignado la totalidad del PV HDD, pero siempre hay que dejar espacio libre por si se necesita.
- La partición EFI como vemos es física, por lo ya explicado, y tiene asignados 100 MB porque es el tamaño recomendado oficial.
- El LV root tiene asignados 30 GB, que está bien para empezar, aunque posiblemente necesite aumento a mitad/final de curso ya que el año pasado mi `/usr` llegó a ocupar unos 60-80 GB.
- `/boot` se encuentra en LVM por razones simples: no daba problemas durante la instalación por tenerlo dentro de LVM como sí dio EFI, y posiblemente necesite aumento de espacio a lo largo del curso cuando empecemos a compilar kernels.
- Con respecto a la swap hay muchos criterios diferentes sobre cuánto debería de ocupar en un sistema, y en mi caso como no suelo hibernar/suspender el portátil, le he asignado un tamaño igual al de mi RAM = 8 GB.
- El LV var es el volumen que con diferencia más espacio necesita para lo que hacemos, así que este año le asigné un tamaño inicial incluso mayor que el año pasado, 170 GB. Incluso con este tamaño, a mitad/final de curso habrá que borrar escenarios de máquinas virtuales para liberar espacio o aumentar el LV, ya que el año pasado llegué a ocupar +300 GB de máquinas virtuales.

Muestro ahora los sistemas de ficheros:

```bash
atlas@olympus:~$ lsblk -f
NAME         FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINT
sda
└─sda1       LVM2_member LVM2 001       K0vDwq-sWWO-9BGf-qniy-pdjj-JmH3-73rk5a
  └─HDD-home xfs                        32345149-c35e-4c4c-8ed7-41e87627c978    215.5G    23% /home
nvme0n1
├─nvme0n1p1  vfat        FAT32          2A00-295A                                89.1M     4% /boot/efi
└─nvme0n1p2  LVM2_member LVM2 001       RXPn3u-nkYK-uiN5-Qncv-AnAf-qZaB-M6ki78
  ├─SSD-root xfs                        4ad9bf47-bcb8-476e-adc3-6c015765ca3e       23G    18% /
  ├─SSD-boot xfs                        814e8a96-bbde-466c-a2e3-73acf67b62fc    843.9M    11% /boot
  ├─SSD-swap swap        1              24e93954-d46f-45ff-b517-93ba486c1692                  [SWAP]
  └─SSD-var  xfs                        392c71c4-6ff6-4435-9931-197de2fdcd21    147.4G     7% /var
```

Como vemos casi todo está en XFS por el dinamismo que aporta este sistema de ficheros, quitando SWAP y EFI que tienen sus propios sistemas de ficheros específicos que necesitan.

## Problemas de drivers

El único problema de drivers que tiene mi portátil *(ASUS TUF Gaming FX504GD)* en Debian, es con el adaptador WiFi. El driver que trae por defecto la instalación de Debian 11 parece no funcionar.

Se soluciona haciendo lo siguiente:

```bash
sudo apt install bc module-assistant build-essential dkms
git clone https://github.com/tomaspinho/rtl8821ce
cd rtl8821ce
sudo m-a prepare
sudo ./dkms-install.sh
```

Después de hacer esto y reiniciar el portátil, la WiFi debería de funcionar correctamente.

Fuente: <https://miloserdov.org/?p=5930>

**¿Qué es lo que se ha hecho aquí realmente?**

Simplemente me he descargado el código fuente del driver específico que necesita mi portátil, y luego lo he compilado e instalado.
