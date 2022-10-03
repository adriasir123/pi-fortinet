# Instalación automática de Debian con preseed

Instrucciones generales:

- Partimos del [preseed.cfg](https://www.debian.org/releases/stable/example-preseed.txt) ejemplo de Debian
- Podemos encontrar la lista de locales posibles en `/usr/share/i18n/SUPPORTED`
- Contraseñas cifradas
    - Instalo: `sudo apt install whois`
    - Cifro: `printf "1234" | mkpasswd -s -m md5`
- LVM con:
    - `/`
    - `/var`
    - `/home`

## Tipo 1: ISO modificada

### 1.1

Usaré la ISO `debian-11.5.0-amd64-netinst.iso` y el siguiente `preseed.cfg`:

```shell
# Locales
d-i debian-installer/language string en_GB:en
d-i debian-installer/country string ES
d-i debian-installer/locale string en_GB.UTF-8


# Keyboard
d-i keyboard-configuration/xkb-keymap select es


# Network
d-i netcfg/choose_interface select auto
d-i netcfg/get_hostname string preseed
d-i netcfg/get_domain string unassigned-domain
d-i netcfg/wireless_wep string


# Mirror
d-i mirror/country string ES
d-i mirror/http/hostname string deb.debian.org
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string
d-i mirror/suite string stable


# Accounts
d-i passwd/root-password-crypted password $1$EuB1h0/P$IIUGcTg79ZeX72l28Fh0E0
d-i passwd/user-fullname string preseed
d-i passwd/username string preseed
d-i passwd/user-password-crypted password $1$N/qFSN1R$Ub33hbzjj4.jBhbygvN0P/


# Clock and time zone
d-i clock-setup/utc boolean true
d-i time/zone string Europe/Madrid
d-i clock-setup/ntp boolean true


# Partitioning
d-i partman-auto/disk string /dev/vda
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-auto/method string lvm
d-i partman-lvm/confirm boolean true
d-i partman-auto/choose_recipe select mypartitioning
d-i partman-auto-lvm/new_vg_name string vg00
d-i partman-auto-lvm/guided_size string max
d-i partman-lvm/confirm_nooverwrite boolean true

d-i partman-auto/expert_recipe string                         \
      mypartitioning ::                                       \
              512 1 512 xfs                                   \
                      $primary{ } $bootable{ }                \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ xfs }     \
                      mountpoint{ /boot }                     \
              .                                               \
              1024 1 1024 linux-swap                          \
                      $defaultignore{ }                       \
                      $lvmok{ }                               \
                      lv_name{ swap }                         \
                      in_vg { vg00 }                          \
                      method{ swap } format{ }                \
              .                                               \
              3072 1 3072 xfs                                 \
                      $defaultignore{ }                       \
                      $lvmok{ }                               \
                      lv_name{ root }                         \
                      in_vg { vg00 }                          \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ xfs }     \
                      mountpoint{ / }                         \
              .                                               \
              6144 1 6144 xfs                                 \
                      $defaultignore{ }                       \
                      $lvmok{ }                               \
                      lv_name{ var }                          \
                      in_vg { vg00 }                          \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ xfs }     \
                      mountpoint{ /var }                      \
              .                                               \
              8192 1 1000000000 xfs                           \
                      $defaultignore{ }                       \
                      $lvmok{ }                               \
                      lv_name{ home }                         \
                      in_vg { vg00 }                          \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ xfs }     \
                      mountpoint{ /home }                     \
              .

d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true


# Base system
d-i base-installer/kernel/image string linux-image-amd64


# Apt
d-i apt-setup/services-select multiselect security, updates
d-i apt-setup/security_host string security.debian.org


# Package selection
tasksel tasksel/first multiselect standard, ssh-server
popularity-contest popularity-contest/participate boolean false


# Boot loader
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true
d-i grub-installer/bootdev  string default


# Finishing up
d-i finish-install/reboot_in_progress note
```

### 1.2

Instalo:

```bash
sudo apt install libarchive-tools
```

Extraigo los ficheros de la ISO en mi directorio `isofiles`:

```bash
bsdtar -C isofiles -xf debian-11.5.0-amd64-netinst.iso
```

### 1.3

Inserto mi `preseed.cfg` en el `initrd`:

```bash
chmod +w -R isofiles/install.amd/
gunzip isofiles/install.amd/initrd.gz
echo preseed.cfg | cpio -H newc -o -A -F isofiles/install.amd/initrd
gzip isofiles/install.amd/initrd
chmod -w -R isofiles/install.amd/
```

### 1.4

Regenero el checksum:

```bash
cd isofiles
chmod +w md5sum.txt
find -follow -type f ! -name md5sum.txt -print0 | xargs -0 md5sum > md5sum.txt
chmod -w md5sum.txt
```

Aparece el siguiente warning, pero lo podemos ignorar:

```bash
find: File system loop detected; ‘./debian’ is part of the same file system loop as ‘.’.
```

### 1.5

Genero la nueva ISO modificada:

```bash
sudo genisoimage -r -J -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o preseed-debian-11.5.0-amd64-netinst.iso .
```

### 1.6

Copio la ISO al pool de imágenes de libvirt para poder crear una máquina a partir de ella:

```bash
sudo cp preseed-debian-11.5.0-amd64-netinst.iso /var/lib/libvirt/images
```

### 1.7

Para que arranque la instalación automática, hacemos lo siguiente:

![arranque1](https://i.imgur.com/fANHtDN.png)

![arranque2](https://i.imgur.com/sXxDyiV.png)

Cuando termine, tendremos la máquina preparada.

### 1.8

Podemos comprobar que el sistema se ha configurado con los datos que hemos proporcionado. Por ejemplo...

Usuario y hostname:

![usuariohostname](https://i.imgur.com/Vuycuh6.png)

Particiones:

![particiones](https://i.imgur.com/NQ3n4X1.png)

Sistemas de ficheros correctos:

![sistemasficheros](https://i.imgur.com/azqAQhI.png)

## Tipo 2: `preseed.cfg` en Nginx

### 2.1

Instalo el servidor web en una VM separada:

```bash
sudo apt install nginx
```

Añado el `preseed.cfg` anterior en la raíz, y muestro que se ha publicado:

![preseedweb](https://i.imgur.com/aFSvLFc.png)

### 2.2

Esta vez crearemos una máquina a partir de la ISO `debian-11.5.0-amd64-netinst.iso` **sin modificar**.

Para que arranque la instalación automática tomando `preseed.cfg` desde Nginx, hacemos lo siguiente:

![arranque1web](https://i.imgur.com/fANHtDN.png)

![arranque2web](https://i.imgur.com/sXxDyiV.png)

![arranque3web](https://i.imgur.com/Pyur3If.png)

Después de hacer esto, la instalación automática se haría.

**¡IMPORTANTE LIMITACIÓN DE D-I EN INSTALACIONES POR RED!**  
El hostname del preseed será ignorado por los motivos explicados [aquí](https://unix.stackexchange.com/questions/106614/preseed-cfg-ignoring-hostname-setting).  
Aún así, existe una solución alternativa, aunque *no es una solución real* al problema ya que no existe.

Tendríamos que añadir lo siguiente al final de `preseed.cfg`:

```bash
# Forced hostname over what DHCP said
d-i preseed/late_command string in-target /bin/bash -c 'echo preseed > /etc/hostname'
```

## Tipo 3: PXE

### 3.1

Instalo el servidor DHCP:

```bash
sudo apt install isc-dhcp-server
```

Dejo `/etc/default/isc-dhcp-server` de la siguiente manera:

```bash
# Defaults for isc-dhcp-server (sourced by /etc/init.d/isc-dhcp-server)

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
#DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPDv4_PID=/var/run/dhcpd.pid
#DHCPDv6_PID=/var/run/dhcpd6.pid

# Additional options to start dhcpd with.
#	Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#	Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="enp1s0"
INTERFACESv6=""
```

Hago una copia de seguridad del fichero de configuración principal original:

```bash
sudo cp dhcpd.conf dhcpd.conf.back
```

Escribo el siguiente `dhcpd.conf`:

```bash
default-lease-time 600;
max-lease-time 7200;

allow booting;

# in this example, we serve DHCP requests from 192.168.0.(3 to 253)
# and we have a router at 192.168.0.1
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.20 192.168.100.30;
  option broadcast-address 192.168.100.255;
  option routers 192.168.100.1;             # our router
  option domain-name-servers 192.168.100.1; # our router has DNS functionality
  next-server 192.168.100.10;                # our Server
  filename "pxelinux.0"; # setting a default, might be wrong for "non defaults"
}
```

Reinicio:

```bash
sudo systemctl restart isc-dhcp-server
```

### 3.2

Instalo el servidor TFTP:

```bash
sudo apt install tftpd-hpa
```

Descargo `netboot.tar.gz` en `/srv/tftp`:

```bash
sudo wget https://deb.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz
```

Extraigo:

```bash
sudo tar -xvf netboot.tar.gz
```

Acabo teniendo la siguiente estructura:

```bash
debian@debian:/srv/tftp$ ls -la
total 16
drwxr-xr-x 3 root root 4096 Oct  3 09:52 .
drwxr-xr-x 3 root root 4096 Oct  3 09:33 ..
drwxr-xr-x 3 root root 4096 Sep  6 22:55 debian-installer
lrwxrwxrwx 1 root root   47 Sep  6 22:55 ldlinux.c32 -> debian-installer/amd64/boot-screens/ldlinux.c32
lrwxrwxrwx 1 root root   33 Sep  6 22:55 pxelinux.0 -> debian-installer/amd64/pxelinux.0
lrwxrwxrwx 1 root root   35 Sep  6 22:55 pxelinux.cfg -> debian-installer/amd64/pxelinux.cfg
-rw-r--r-- 1 root root   65 Sep  6 22:55 version.info
```

Reinicio:

```bash
sudo systemctl restart tftpd-hpa
```

### 3.3

Para crear una VM que bootee desde PXE sin ISO, hago los siguientes pasos:

![arranque1pxe](https://i.imgur.com/orVseXc.png)

![arranque2pxe](https://i.imgur.com/9xBVS8q.png)

![arranque3pxe](https://i.imgur.com/K90t1lZ.png)

![arranque4pxe](https://i.imgur.com/gLbmPzO.png)

![arranque5pxe](https://i.imgur.com/Tf8Bp4f.png)

![arranque6pxe](https://i.imgur.com/0qxD0F8.png)

Conecto la VM a una red creada por mí, sin DHCP, para que el servidor PXE sea el único con DHCP en la red

![arranque7pxe](https://i.imgur.com/qRkTtzJ.png)

Al principio el arranque falla, pero es porque necesitamos modificarlo en View > Details

![arranque8pxe](https://i.imgur.com/5E4MEPS.png)

Reiniciamos la máquina, y el arranque por PXE funcionará:

![arranque9pxe](https://i.imgur.com/6VpZ61F.png)

![arranque10pxe](https://i.imgur.com/nMZZOz4.png)

Usamos de nuevo el método de carga de `preseed.cfg` por Nginx. Esta vez instalado en el servidor PXE:

![arranque11pxe](https://i.imgur.com/Eu161UL.png)

Terminada la instalación, volvemos a cambiar el orden de arranque:

![arranque12pxe](https://i.imgur.com/znFZaQi.png)

Vemos que tenemos la VM funcionando, y con una IP dentro del pool:

![arranque13pxe](https://i.imgur.com/L9qm1gZ.png)
