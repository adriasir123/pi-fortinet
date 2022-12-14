---
title: "Tarea: Creación de un sistema automatizado de instalación"
---

# Enunciado

Implementa un sistema de instalación con PXE/TFTP que realice una instalación completa de un sistema Debian Buster de forma totalmente desatendida, pasándole todos los parámetros necesarios por "preseeding"


# Planteamiento/Organización

1 VMs en red interna Vagrant

1 SERVER (imagen debian buster + fichero preseeding + server PXE + server DHCP + TFTP server)

2 CLIENT (booteo con PXE, pedirá una IP al DHCP, se instalará el sistema localmente a través del servidor) (MÁQUINA VACÍA VIRTUALBOX PARA QUE NO INSTALE NADA EN EL DISCO DURO)

(probar a poner por PXE el cliente en modo bridge, y que acceda al PXE del gonzalo nazareno) (no se ha podido porque no había imágenes booteables, no funciona el PXE de aquí)

https://wiki.debian.org/PXEBootInstall

# Desarrollo

## 1. Configuración del servidor DHCP (servidor)

```
sudo apt install isc-dhcp-server
```

En `/etc/default/isc-dhcp-server`
```
INTERFACESv4="eth1"
```
Decimos por qué interfaces servimos peticiones DHCP


En `/etc/dhcp/dhcpd.conf` (propia configuración de DHCP)
```
default-lease-time 600;
max-lease-time 7200;

allow booting;
allow bootp;

# The next paragraph needs to be modified to fit your case
subnet 192.168.0.0 netmask 255.255.255.0 {
  range 192.168.0.3 192.168.0.10;
  option broadcast-address 192.168.0.255;
# the gateway address which can be different
# (access to the internet for instance)
#  option routers 192.168.1.1;
# indicate the dns you want to use
#  option domain-name-servers 192.168.1.3;
}

group {
  next-server 192.168.0.2;
  host tftpclient {
# tftp client hardware address
  hardware ethernet  08:00:27:3a:73:29;
  filename "pxelinux.0";
 }
}
```
> Esta configuración sólo serviría para el arranque por PXE de una máquina concreta, porque si se conectase otra cualquiera no podría. 


Reiniciar servicio para que tenga en cuenta la configuración
```
sudo systemctl restart isc-dhcp-server
```
Comprobar que se está ejecutando
```
sudo systemctl status isc-dhcp-server
```

## 2. Configuración del servidor TFTP (servidor)

```
sudo apt install tftpd-hpa
```

> https://www.debian.org/releases/jessie/amd64/ch04s05.html.en
> Importante enlace para que funcione el booteo desde el cliente, por la configuración del DHCP básicamente

## 3. Proporcionar la imagen de instalación (servidor)

Nos posicionamos en el directorio de tftp
```
cd /srv/tftp/
```

Aquí, obtenemos la imagen de instalación por red
```
sudo wget http://ftp.nl.debian.org/debian/dists/buster/main/installer-amd64/current/images/netboot/netboot.tar.gz
```

Descomprimimos y borramos el comprimido original para mayor limpieza
```
sudo tar -xzf netboot.tar.gz
sudo rm netboot.tar.gz
```

Tenemos la siguiente estructura
```
drwxrwxr-x 3 root root 4096 Oct 16 09:01 .
drwxr-xr-x 3 root root 4096 Oct 16 08:30 ..
drwxrwxr-x 3 root root 4096 Sep 20 23:11 debian-installer
lrwxrwxrwx 1 root root   47 Sep 20 23:11 ldlinux.c32 -> debian-installer/amd64/boot-screens/ldlinux.c32
lrwxrwxrwx 1 root root   33 Sep 20 23:11 pxelinux.0 -> debian-installer/amd64/pxelinux.0
lrwxrwxrwx 1 root root   35 Sep 20 23:11 pxelinux.cfg -> debian-installer/amd64/pxelinux.cfg
-rw-rw-r-- 1 root root   63 Sep 20 23:11 version.info
```

Reiniciamos el servicio
```
sudo systemctl restart tftpd-hpa.service
```


## 4. Comprobación de que el cliente arranca correctamente el instalador por red

![](https://i.imgur.com/gD1w9WO.png)


## 5. Preseeding, instalación automática

Fichero `preseed.cfg` en `/srv/tftp/`
```
# Services to use:
# Choices: security updates (from security.debian.org), release updates, backported software
apt-setup-udeb	apt-setup/services-select	multiselect	security, updates
# for internal use; can be preseeded
d-i	preseed/run/checksum	string	
# location
# Choices: Auckland, Chatham Islands
tzsetup-udeb	tzsetup/country/NZ	select	Pacific/Auckland
# Allow login as root?
user-setup-udeb	passwd/root-login	boolean	false
# Device for boot loader installation:
grub-installer	grub-installer/bootdev	string	
# for internal use; can be preseeded
pkgsel	pkgsel/updatedb	boolean	true
# for internal use only
bootstrap-base	base-installer/kernel/linux/initrd-2.6	boolean	true
# Bad archive mirror
d-i	mirror/bad	error	
# for internal use only
d-i	debconf/language	string	en
# This is an overview of your currently configured partitions and mount points. Select a partition to modify its settings (file system, mount point, etc.), a free space to create partitions, or a device to initialize its partition table.
# Choices: Guided partitioning, Configure software RAID, Configure the Logical Volume Manager, Configure encrypted volumes, Configure iSCSI volumes, , SCSI3 (0\,0\,0) (sda) - 8.6 GB ATA VBOX HARDDISK, >   ${!TAB}${!ALIGN=RIGHT}#1${!TAB}primary${!TAB}${!ALIGN=RIGHT}8.1 GB${!TAB}${!TAB}f${!TAB}ext4${!TAB}${!TAB}/${!TAB}, >   ${!TAB}${!ALIGN=RIGHT}#5${!TAB}logical${!TAB}${!ALIGN=RIGHT}534.8 MB${!TAB}${!TAB}f${!TAB}swap${!TAB}${!TAB}swap${!TAB}, , Undo changes to partitions, Finish partitioning and write changes to disk
partman-base	partman/choose_partition	select	90finish__________finish
# 
# Choices: American Samoa, Australia, Cook Islands, Fiji, French Polynesia, Guam, Kiribati, Marshall Islands, Micronesia\, Federated States of, Nauru, New Caledonia, New Zealand, Niue, Norfolk Island, Northern Mariana Islands, Palau, Papua New Guinea, Pitcairn, Samoa, Solomon Islands, Tokelau, Tonga, Tuvalu, United States Minor Outlying Islands, Vanuatu, Wallis and Futuna
d-i	localechooser/countrylist/Oceania	select	
# Full name for the new user:
user-setup-udeb	passwd/user-fullname	string	pxe-client-user
# for internal use; can be preseeded
d-i	debian-installer/allow_unauthenticated_ssl	boolean	false
# for internal use
d-i	keyboard-configuration/variantcode	string	
# Tool to use to generate boot initrd:
# Choices: 
bootstrap-base	base-installer/initramfs/generator	select	
# Encryption configuration failure
partman-crypto	partman-crypto/crypto_root_needs_boot	error	
#  is too big
partman-auto-lvm	partman-auto-lvm/big_guided_size	error	
# Unreachable gateway
d-i	netcfg/gateway_unreachable	error	
# for internal use; can be preseeded
apt-setup-udeb	apt-setup/multiarch	string	
# location
# Choices: Godthab, Danmarkshavn, Scoresbysund, Thule
tzsetup-udeb	tzsetup/country/GL	select	America/Godthab
# Debootstrap Error
bootstrap-base	base-installer/debootstrap/error/nogetrel	error	
# Device for boot loader installation:
# Choices: Enter device manually, /dev/sda  (ata-VBOX_HARDDISK_VBaeba5a60-2d9f1466)
grub-installer	grub-installer/choose_bootdev	select	/dev/sda
# Failed to mount the floppy
d-i	save-logs/floppy_mount_failed	error	
# Country, territory or area:
# Choices: Albania, Andorra, Armenia, Austria, Azerbaijan, Belarus, Belgium, Bosnia and Herzegovina, Bulgaria, Croatia, Cyprus, Czechia, Denmark, Estonia, Faroe Islands, Finland, France, Georgia, Germany, Gibraltar, Greece, Greenland, Guernsey, Holy See (Vatican City State), Hungary, Iceland, Ireland, Isle of Man, Italy, Jersey, Latvia, Liechtenstein, Lithuania, Luxembourg, Macedonia\, Republic of, Malta, Moldova, Monaco, Montenegro, Netherlands, Norway, Poland, Portugal, Romania, Russian Federation, San Marino, Serbia, Slovakia, Slovenia, Spain, Svalbard and Jan Mayen, Sweden, Switzerland, Ukraine, United Kingdom, Åland Islands
d-i	localechooser/countrylist/Europe	select	ES
# 
# Choices: Spain, France, other
d-i	localechooser/shortlist/eu	select	
# Cannot save logs
d-i	save-logs/bad_directory	error	
# for internal use; can be preseeded
d-i	netcfg/disable_autoconfig	boolean	false
# Entering low memory mode
d-i	lowmem/low	note	
# Location of initial preconfiguration file:
d-i	preseed/url	string	
# city
# Choices: Almaty, Qyzylorda, Aqtobe, Atyrau, Oral
tzsetup-udeb	tzsetup/country/KZ	select	Asia/Almaty
# Netmask:
d-i	netcfg/get_netmask	string	
# Ignore questions with a priority less than:
# Choices: critical, high, medium, low
d-i	debconf/priority	select	low
# for internal use; can be preseeded
bootstrap-base	base-installer/kernel/linux/extra-packages-2.6	string	
# Volume group name already in use
partman-lvm	partman-lvm/vgcreate_nameused	error	
# btrfs root file system not supported without separate /boot
partman-btrfs	partman-btrfs/btrfs_root	error	
# Empty password
user-setup-udeb	user-setup/password-empty	error	
# country code or "manual" (for internal use)
d-i	mirror/country	string	ES
# Error while creating a new logical volume
partman-lvm	partman-lvm/lvcreate_error	error	
# Current LVM configuration:
partman-lvm	partman-lvm/displayall	note	
# for internal use only
d-i	debconf/showold	boolean	false
# for internal use only
user-setup-udeb	passwd/user-uid	string	
# Error
d-i	netcfg/error	error	
# Use contrib software?
apt-mirror-setup	apt-setup/contrib	boolean	false
# Setting firmware variables for automatic boot
nobootloader	nobootloader/confirmation_powerpc_pasemi	note	
# No root file system
partman-target	partman-target/no_root	error	
# Install the GRUB boot loader to the Serial ATA RAID disk?
grub-installer	grub-installer/sataraid	boolean	true
# zone
# Choices: Eastern, Central, Mountain, Pacific, Alaska, Hawaii, Arizona, East Indiana, Samoa
tzsetup-udeb	tzsetup/country/US	select	US/Eastern
# Name of the volume group for the new system:
partman-auto-lvm	partman-auto-lvm/new_vg_name	string	
# Debootstrap warning
bootstrap-base	base-installer/debootstrap/fallback-warning	error	
# Name server addresses:
d-i	netcfg/get_nameservers	string	
# Translations temporarily not available
d-i	localechooser/translation/none-yet	note	
# The partition starts from  and ends at .
partman-base	partman/show_partition_chs	note	
# 
# Choices: Aruba, Belgium, Netherlands, other
d-i	localechooser/shortlist/nl	select	
# 
# Choices: 
partman-base	partman/exception_handler	select	
# Debian archive mirror:
# Choices: ftp.es.debian.org, ulises.hostalia.com, deb.debian.org, debian-archive.trafficmanager.net, softlibre.unizar.es, debian.redparra.com, debian.grn.cat, ftp.udc.es, ftp.cica.es, ftp.caliu.cat, debian.redimadrid.es, debian.uvigo.es, mirror.librelabucm.org
d-i	mirror/http/mirror	select	deb.debian.org
# Password input error
grub-installer	grub-installer/password-mismatch	error	
# Failed to mount /target/proc
nobootloader	nobootloader/mounterr	error	
# for internal use; can be preseeded
grub-installer	grub-installer/skip	boolean	false
# No network interfaces detected
d-i	netcfg/no_interfaces	error	
# Keep default keyboard layout ()?
d-i	keyboard-configuration/unsupported_layout	boolean	true
# 
# Choices: Finland, Sweden, other
d-i	localechooser/shortlist/sv	select	
# Continue the installation in the selected language?
d-i	localechooser/translation/warn-light	boolean	true
# Unable to install the selected kernel
bootstrap-base	base-installer/kernel/failed-install	error	
# location
# Choices: Tahiti (Society Islands), Marquesas Islands, Gambier Islands
tzsetup-udeb	tzsetup/country/PF	select	Pacific/Tahiti
# Amount of volume group to use for guided partitioning:
partman-auto-lvm	partman-auto-lvm/guided_size	string	some number
# for internal use; can be preseeded
partman-iscsi	partman-iscsi/login/all_targets	boolean	false
# Configuration of encrypted volumes failed
partman-crypto	partman-crypto/commit_failed	error	
# Choose an installation step:
# Choices: Partition disks
d-i	debian-installer/missing-provide	select	Partition disks
# Continue without a network mirror?
apt-mirror-setup	apt-setup/no_mirror	boolean	false
# Network configuration method:
# Choices: Retry network autoconfiguration, Retry network autoconfiguration with a DHCP hostname, Configure network manually, , Do not configure the network at this time
d-i	netcfg/dhcp_options	select	Configure network manually
# iSCSI targets on :
# Choices: 
partman-iscsi	partman-iscsi/login/targets	multiselect	
# btrfs file system not supported for /boot
partman-btrfs	partman-btrfs/btrfs_boot	error	
# No software RAID devices available
partman-md	partman-md/delete_no_md	error	
# Failed to delete the software RAID device
partman-md	partman-md/deletefailed	error	
# for internal use; can be preseeded
disk-detect	disk-detect/multipath/enable	boolean	false
# Mount point for this partition:
# Choices: /dos, /windows, Enter manually, Do not mount it
partman-basicfilesystems	partman-basicfilesystems/fat_mountpoint	select	
# for internal use; can be preseeded
d-i	preseed/url/checksum	string	
# Really delete the volume group?
partman-lvm	partman-lvm/vgdelete_confirm	boolean	true
# 
# Choices: Algeria, Angola, Benin, Botswana, Burkina Faso, Burundi, Cabo Verde, Cameroon, Central African Republic, Chad, Congo, Congo\, The Democratic Republic of the, Côte d'Ivoire, Djibouti, Egypt, Equatorial Guinea, Eritrea, Ethiopia, Gabon, Gambia, Ghana, Guinea, Guinea-Bissau, Kenya, Lesotho, Liberia, Libya, Malawi, Mali, Mauritania, Morocco, Mozambique, Namibia, Niger, Nigeria, Rwanda, Sao Tome and Principe, Senegal, Sierra Leone, Somalia, South Africa, South Sudan, Sudan, Swaziland, Tanzania, Togo, Tunisia, Uganda, Western Sahara, Zambia, Zimbabwe
d-i	localechooser/countrylist/Africa	select	
# Downloading a file failed:
# Choices: Retry, Change mirror, Ignore
apt-mirror-setup	apt-setup/mirror/error	select	Retry
# Debootstrap Error
bootstrap-base	base-installer/debootstrap/error/unknownrelsig	error	
# for internal use; can be preseeded
d-i	debian-installer/framebuffer	boolean	false
# 
# Choices: British Indian Ocean Territory, Christmas Island, Cocos (Keeling) Islands, Comoros, French Southern Territories, Heard Island and McDonald Islands, Madagascar, Maldives, Mauritius, Mayotte, Réunion, Seychelles
d-i	localechooser/countrylist/Indian_Ocean	select	
# No iSCSI targets discovered
partman-iscsi	partman-iscsi/login/no_targets	error	
# Debian archive mirror hostname:
d-i	mirror/https/hostname	string	mirror
# for internal use; can be preseeded
partman-auto	partman-auto/expert_recipe	string	
# iSCSI configuration actions
# Choices: Log into iSCSI targets, Finish
partman-iscsi	partman-iscsi/mainmenu	select	
# for internal use only
d-i	debconf/translations-dropped	boolean	false
# Continue the installation in the selected language?
d-i	localechooser/translation/warn-severe	boolean	false
# Cannot install kernel
bootstrap-base	base-installer/kernel/no-kernels-found	error	
# Invalid username
user-setup-udeb	passwd/username-bad	error	
#  is too small
partman-auto-lvm	partman-auto-lvm/small_guided_size	error	
# No file system mounted on /target
base-installer	base-installer/no_target_mounted	error	
# Name of the volume group for the new system:
partman-auto-lvm	partman-auto-lvm/new_vg_name_exists	string	
# Non-existing physical volume
partman-auto-lvm	partman-auto-lvm/no_such_pv	error	
# 
# Choices: China, India, other
d-i	localechooser/shortlist/bo	select	
# for internal use; can be preseeded
grub-installer	grub-installer/make_active	boolean	true
# The size entered is too large
partman-partitioning	partman-partitioning/big_new_size	error	
# Layout of the RAID10 array:
partman-md	partman-md/raid10layout	string	n2
# for internal use; can be preseeded
d-i	preseed/run	string	
# Debian archive mirror directory:
d-i	mirror/https/directory	string	/debian/
# Failed to create a swap space
partman-basicfilesystems	partman-basicfilesystems/create_swap_failed	error	
# Go back to the menu?
partman-basicmethods	partman-basicmethods/method_only	boolean	
# Ethernet card not found
d-i	ethdetect/cannot_find	error	
# zone
# Choices: Moscow-01 - Kaliningrad, Moscow+00 - Moscow, Moscow+01 - Samara, Moscow+02 - Yekaterinburg, Moscow+03 - Omsk, Moscow+04 - Krasnoyarsk, Moscow+05 - Irkutsk, Moscow+06 - Yakutsk, Moscow+07 - Vladivostok, Moscow+08 - Magadan, Moscow+09 - Kamchatka
tzsetup-udeb	tzsetup/country/RU	select	Europe/Moscow
# for internal use only
d-i	mirror/codename	string	buster
# Volume group name:
partman-lvm	partman-lvm/vgcreate_name	string	
# zone
# Choices: Tarawa (Gilbert Islands), Enderbury (Phoenix Islands), Kiritimati (Line Islands)
tzsetup-udeb	tzsetup/country/KI	select	Pacific/Tarawa
# Go back to the menu and correct this problem?
partman-ext3	partman-ext3/boot_not_ext2_or_ext3	boolean	
# Keyboard model:
# Choices: 
d-i	keyboard-configuration/model	select	
# Load missing firmware from removable media?
d-i	hw-detect/load_firmware	boolean	true
# Set the clock using NTP?
clock-setup	clock-setup/ntp	boolean	true
# Invalid passphrase
d-i	netcfg/invalid_pass	error	
# Location for the new partition:
# Choices: Beginning, End
partman-partitioning	partman-partitioning/new_partition_place	select	
# Do you want to resume partitioning?
partman-target	partman-target/mount_failed	boolean	true
# LILO installation failed. Continue anyway?
lilo-installer	lilo-installer/apt-install-failed	boolean	true
# Do you want to return to the partitioning menu?
partman-basicfilesystems	partman-basicfilesystems/no_swap	boolean	true
# No devices selected
partman-crypto	partman-crypto/create/nosel	error	
# LILO installation target:
# Choices: : software RAID array, Other choice (Advanced)
lilo-installer	lilo-installer/bootdev_raid	select	
# Error while setting up RAID
partman-auto-raid	partman-auto-raid/error	error	
# for internal use; can be preseeded
d-i	auto-install/defaultroot	string	d-i/buster/./preseed.cfg
# Devices for the new volume group:
# Choices: 
partman-lvm	partman-lvm/vgcreate_parts	multiselect	
# Do you intend to use FireWire Ethernet?
d-i	ethdetect/use_firewire_ethernet	boolean	false
# Base system installation error
bootstrap-base	base-installer/debootstrap/error-exitcode	error	
# LILO installation target:
lilo-installer	lilo-installer/manual_bootdev	string	
# Key to function as AltGr:
# Choices: The default for the keyboard layout, No AltGr key, Right Alt (AltGr), Right Control, Right Logo key, Menu key, Left Alt, Left Logo key, Keypad Enter key, Both Logo keys, Both Alt keys
d-i	keyboard-configuration/altgr	select	The default for the keyboard layout
# Web server started, but network not running
d-i	save-logs/no_network	note	
# Debootstrap Error
bootstrap-base	base-installer/debootstrap/fallback-error	error	
# No boot loader installed
nobootloader	nobootloader/confirmation_common	note	
# Debootstrap Error
bootstrap-base	base-installer/debootstrap/error/invalidrel	error	
# for internal use only
d-i	debian-installer/language	string	en
# LILO configured to use a serial console
lilo-installer	lilo-installer/serial-console	note	
# Continue with partitioning?
partman-partitioning	partman-partitioning/unknown_label	boolean	true
# 
# Choices: South Sudan, Jordan, United Arab Emirates, Bahrain, Algeria, Syrian Arab Republic, Saudi Arabia, Sudan, Iraq, Kuwait, Morocco, India, Yemen, Tunisia, Oman, Qatar, Lebanon, Libya, Egypt, other
d-i	localechooser/shortlist/ar	select	
# Required programs missing
partman-crypto	partman-crypto/tools_missing	note	
# Unable to configure GRUB
grub-installer	grub-installer/multipath-error	error	
# zone
# Choices: Santiago, Easter Island
tzsetup-udeb	tzsetup/country/CL	select	America/Santiago
# for internal use; can be preseeded
d-i	preseed/include_command	string	
# for internal use; can be preseeded
d-i	debian-installer/theme	string	
# Create new empty partition table on this device?
partman-partitioning	partman-partitioning/confirm_new_label	boolean	false
# 
# Choices: Greece, Cyprus, other
d-i	localechooser/shortlist/el	select	
# Partition table type:
# Choices: atari, aix, amiga, bsd, dvh, gpt, mac, msdos, pc98, sun, loop
partman-partitioning	partman-partitioning/choose_label	select	
# for internal use
d-i	keyboard-configuration/store_defaults_in_debconf_db	boolean	true
# Hostname:
d-i	netcfg/get_hostname	string	pxe-client
# for internal use; can be preseeded
bootstrap-base	base-installer/kernel/linux/extra-packages	string	
# No volume group found
partman-lvm	partman-lvm/lvcreate_nofreevg	error	
# iSCSI target portal address:
partman-iscsi	partman-iscsi/login/address	string	
# for internal use; can be preseeded
d-i	ethdetect/prompt_missing_firmware	boolean	true
# Volume group to extend:
# Choices: 
partman-lvm	partman-lvm/vgextend_names	select	
# Proceed to install crypto components despite insufficient memory?
partman-crypto	partman-crypto/install_udebs_low_mem	boolean	
# No volume group found
partman-lvm	partman-lvm/vgdelete_novg	error	
# for internal use; can be preseeded
d-i	rescue/enable	boolean	false
# Type of encryption key for this partition:
# Choices: 
partman-crypto	partman-crypto/keytype	select	
# New partition size:
partman-partitioning	partman-partitioning/new_size	string	some number
# Software RAID device type:
# Choices: RAID0, RAID1, RAID5, RAID6, RAID10
partman-md	partman-md/createmain	select	
# Active devices for the RAID0 array:
# Choices: 
partman-md	partman-md/raid0devs	multiselect	
# Unusable free space
partman-auto	partman-auto/unusable_space	error	
# Failed to download crypto components
partman-crypto	partman-crypto/install_udebs_failure	error	
# Error while extending volume group
partman-lvm	partman-lvm/vgextend_error	error	
# Keep current partition layout and configure RAID?
partman-md	partman-md/confirm_nochanges	boolean	false
# HTTP proxy information (blank for none):
d-i	mirror/http/proxy	string	
# Install GRUB?
grub-installer	grub-installer/grub_not_mature_on_this_platform	boolean	false
# Software RAID not available
partman-md	partman-md/nomd	error	
# Erasing data on  failed
partman-crypto	partman-crypto/plain_erase_failed	error	
# iSCSI target username for :
partman-iscsi	partman-iscsi/login/incoming_username	string	
# No RAID partitions available
partman-md	partman-md/noparts	error	
# Select disk to partition:
# Choices: SCSI3 (0\,0\,0) (sda) - 8.6 GB ATA VBOX HARDDISK
partman-auto	partman-auto/select_disk	select	/var/lib/partman/devices/=dev=sda
# Debian archive mirror hostname:
d-i	mirror/http/hostname	string	deb.debian.org
# Separate file system not allowed here
partman-target	partman-target/must_be_on_root	error	
# for internal use; can be preseeded
partman-auto-crypto	partman-auto-crypto/erase_disks	boolean	true
# Volume group to delete:
# Choices: 
partman-lvm	partman-lvm/vgdelete_names	select	
# Really erase the data on ?
partman-crypto	partman-crypto/plain_warn_erase	boolean	false
# for internal use; can be preseeded
d-i	preseed/file	string	
# HTTP proxy information (blank for none):
d-i	mirror/https/proxy	string	
# Empty passphrase
partman-crypto	partman-crypto/passphrase-empty	error	
# No volume group found
partman-lvm	partman-lvm/vgextend_novg	error	
# Primary network interface:
# Choices: enp0s3: Intel Corporation 82540EM Gigabit Ethernet Controller, enp0s8: Intel Corporation 82540EM Gigabit Ethernet Controller
d-i	netcfg/choose_interface	select	enp0s8: Intel Corporation 82540EM Gigabit Ethernet Controller
# Encryption configuration actions
# Choices: Create encrypted volumes, Finish
partman-crypto	partman-crypto/mainmenu	select	
# Mount options:
# Choices: 
partman-basicfilesystems	partman-basicfilesystems/mountoptions	multiselect	
# zone
# Choices: Newfoundland, Atlantic, Eastern, Central, East Saskatchewan, Saskatchewan, Mountain, Pacific
tzsetup-udeb	tzsetup/country/CA	select	Canada/Eastern
# Partitioning method:
# Choices: 
partman-auto	partman-auto/automatically_partition	select	
# Mount point for this partition:
partman-basicfilesystems	partman-basicfilesystems/mountpoint_manual	string	
# Logical volume size:
partman-lvm	partman-lvm/lvcreate_size	string	
# 
# Choices: India, Sri Lanka, other
d-i	localechooser/shortlist/ta	select	
# Keep current partition layout and configure encrypted volumes?
partman-crypto	partman-crypto/confirm_nochanges	boolean	false
# Are you sure you want a bootable logical partition?
partman-partitioning	partman-partitioning/bootable_logical	boolean	false
# Reserved username
user-setup-udeb	passwd/username-reserved	error	
# for internal use only
d-i	anna/retriever	string	net-retriever
# Write the changes to disks and configure LVM?
d-i	partman-lvm/confirm_nooverwrite	boolean	false
# 
# Choices: Russian Federation, Ukraine, other
d-i	localechooser/shortlist/ru	select	
# Gateway:
d-i	netcfg/get_gateway	string	
# The size entered is too small
partman-partitioning	partman-partitioning/small_new_size	error	
# 
d-i	debian-installer/shell-plugin	terminal	
# The free space starts from  and ends at .
partman-base	partman/show_free_chs	note	
# Use non-free software?
apt-mirror-setup	apt-setup/non-free	boolean	false
# No physical volumes selected
partman-lvm	partman-lvm/vgextend_nosel	error	
# Go back to the menu and correct this problem?
partman-ext3	partman/boot_not_first_partition	boolean	
# Invalid ESSID
d-i	netcfg/invalid_essid	error	
# Do you want to return to the partitioning menu?
partman-ext3	partman-ext3/bad_alignment	boolean	
# 
# Choices: China, Singapore, Taiwan, Hong Kong, other
d-i	localechooser/shortlist/zh_TW	select	
# Invalid network link detection waiting time
d-i	netcfg/bad_link_wait_timeout	error	
# location
# Choices: Lisbon, Madeira Islands, Azores
tzsetup-udeb	tzsetup/country/PT	select	Europe/Lisbon
# Invalid WEP key
d-i	netcfg/invalid_wep	error	
# New partition size:
partman-partitioning	partman-partitioning/new_partition_size	string	some number
# Write the changes to disks and configure LVM?
partman-lvm	partman-lvm/confirm	boolean	false
# zone
# Choices: North-West, Pacific, Sonora, Central, South-East
tzsetup-udeb	tzsetup/country/MX	select	Mexico/General
# Network autoconfiguration failed
d-i	netcfg/dhcp_failed	note	
# for internal use; can be preseeded
d-i	netcfg/dhcpv6_timeout	string	15
# Failed to process the preconfiguration file
d-i	preseed/load_error	error	
# Write the changes to disk and configure encrypted volumes?
partman-crypto	partman-crypto/confirm	boolean	false
# for internal use; can be preseeded
pkgsel	pkgsel/include	string	
# Use Control+Alt+Backspace to terminate the X server?
d-i	keyboard-configuration/ctrl_alt_bksp	boolean	false
# 
# Choices: Serbia, Montenegro, other
d-i	localechooser/shortlist/sr	select	
# The encryption key for  is now being created.
partman-crypto	partman-crypto/entropy	entropy	
# Interactive shell
d-i	di-utils-shell/do-shell	note	
# Downloading local repository key failed:
# Choices: Retry, Ignore
apt-setup-udeb	apt-setup/local/key-error	select	Retry
# Debootstrap Error
bootstrap-base	base-installer/no_codename	error	
# Partition in use
partman-base	partman-base/partlocked	error	
# Load missing drivers from removable media?
d-i	hw-detect/load_media	boolean	false
# Continue without a default route?
d-i	netcfg/no_default_route	boolean	false
# Go back to the menu and correct errors?
partman-basicfilesystems	partman-basicfilesystems/swap_check_failed	boolean	
# Spare devices for the RAID array:
# Choices: 
partman-md	partman-md/raidsparedevs	multiselect	
# Auto-configure networking?
d-i	netcfg/use_autoconfig	boolean	true
# Language:
# Choices: C${!TAB}-${!TAB}No localization, Basque${!TAB}-${!TAB}Euskara, Catalan${!TAB}-${!TAB}Català, Danish${!TAB}-${!TAB}Dansk, Dutch${!TAB}-${!TAB}Nederlands, English${!TAB}-${!TAB}English, Finnish${!TAB}-${!TAB}Suomi, French${!TAB}-${!TAB}Français, Galician${!TAB}-${!TAB}Galego, German${!TAB}-${!TAB}Deutsch, Icelandic${!TAB}-${!TAB}Íslenska, Indonesian${!TAB}-${!TAB}Bahasa Indonesia, Irish${!TAB}-${!TAB}Gaeilge, Italian${!TAB}-${!TAB}Italiano, Northern Sami${!TAB}-${!TAB}Sámegillii, Norwegian Bokmaal${!TAB}-${!TAB}Norsk bokmål, Norwegian Nynorsk${!TAB}-${!TAB}Norsk nynorsk, Portuguese${!TAB}-${!TAB}Português, Portuguese (Brazil)${!TAB}-${!TAB}Português do Brasil, Spanish${!TAB}-${!TAB}Español, Swedish${!TAB}-${!TAB}Svenska, Tagalog${!TAB}-${!TAB}Tagalog
d-i	localechooser/languagelist	select	C
# Key size for this partition:
# Choices: 
partman-crypto	partman-crypto/keysize	select	
# Force UEFI installation?
partman-efi	partman-efi/non_efi_system	boolean	
# for internal use
bootstrap-base	base-installer/kernel/linux/initramfs-tools/driver-policy	string	most
# Error while reducing volume group
partman-lvm	partman-lvm/vgreduce_error	error	
# for internal use
d-i	keyboard-configuration/modelcode	string	pc105
# Number of active devices for the RAID array:
partman-md	partman-md/raiddevcount	string	
# Failed to create a file system
partman-basicfilesystems	partman-basicfilesystems/create_failed	error	
# Failure of key exchange and association
d-i	netcfg/wpa_supplicant_failed	note	
# Go back to the menu and resume partitioning?
partman-efi	partman-efi/no_efi	boolean	
# Resize operation failure
partman-partitioning	partman-partitioning/new_size_commit_failed	error	
# Protocol for file downloads:
# Choices: http, https, ftp
d-i	mirror/protocol	select	http
# WPA/WPA2 passphrase for wireless device :
d-i	netcfg/wireless_wpa	string	
# for internal use only
d-i	debian-installer/exit/always_halt	boolean	false
# Cannot install base system
bootstrap-base	base-installer/cannot_install	error	
# Keep current partition layout and configure LVM?
partman-lvm	partman-lvm/confirm_nochanges	boolean	false
# IPv6 unsupported on point-to-point links
d-i	netcfg/no_ipv6_pointopoint	error	
# Identical mount points for two file systems
partman-target	partman-target/same_mountpoint	error	
# for internal use; can be preseeded
d-i	preseed/early_command	string	
# for internal use only
partman-auto-raid	partman-auto-raid/raidnum	string	
# 
# Choices: China, Taiwan, Singapore, Hong Kong, other
d-i	localechooser/shortlist/zh_CN	select	
# Insert formatted floppy in drive
d-i	save-logs/insert_floppy	note	
# for internal use
d-i	keyboard-configuration/optionscode	string	
# Debian archive mirror directory:
d-i	mirror/ftp/directory	string	/debian/
# Debian version to install:
# Choices: buster${!TAB}-${!TAB}stable, bullseye${!TAB}-${!TAB}testing, sid${!TAB}-${!TAB}unstable
d-i	mirror/suite	select	stable
# Unable to configure GRUB
grub-installer	grub-installer/sataraid-error	error	
# FTP proxy information (blank for none):
d-i	mirror/ftp/proxy	string	
# for internal use; can be preseeded
d-i	netcfg/enable	boolean	true
# Additional parameters for module :
d-i	hw-detect/retry_params	string	
# for internal use; can be preseeded
bootstrap-base	base-installer/includes	string	
# Point-to-point address:
d-i	netcfg/get_pointopoint	string	
# Failed to mount /target/proc
grub-installer	grub-installer/mounterr	error	
# 
# Choices: Italy, Switzerland, other
d-i	localechooser/shortlist/it	select	
# Method for temporarily toggling between national and Latin input:
# Choices: No temporary switch, Both Logo keys, Right Alt (AltGr), Right Logo key, Left Alt, Left Logo key
d-i	keyboard-configuration/switch	select	No temporary switch
# 
# Choices: Anguilla, Antigua and Barbuda, Aruba, Bahamas, Barbados, Bermuda, Bonaire\, Sint Eustatius and Saba, Cayman Islands, Cuba, Dominica, Dominican Republic, Grenada, Guadeloupe, Haiti, Jamaica, Martinique, Montserrat, Puerto Rico, Saint Barthélemy, Saint Kitts and Nevis, Saint Lucia, Saint Martin (French part), Saint Vincent and the Grenadines, Sint Maarten (Dutch part), Trinidad and Tobago, Turks and Caicos Islands, Virgin Islands\, British, Virgin Islands\, U.S.
d-i	localechooser/countrylist/Caribbean	select	
# Help on partitioning
partman-target	partman-target/help	note	
# Base system installation error
bootstrap-base	base-installer/debootstrap/error-abnormal	error	
# for internal use only
user-setup-udeb	passwd/user-default-groups	string	audio cdrom dip floppy video plugdev netdev scanner bluetooth debian-tor lpadmin
# Volume group:
# Choices: 
partman-lvm	partman-lvm/lvcreate_vgnames	select	
# Keyboard layout:
# Choices: 
d-i	keyboard-configuration/variant	select	
# 
# Choices: Belgium, Canada, France, Luxembourg, Switzerland, other
d-i	localechooser/shortlist/fr	select	
# Software RAID configuration actions
# Choices: Create MD device, Delete MD device, Finish
partman-md	partman-md/mainmenu	select	
# Logical volume name:
partman-lvm	partman-lvm/lvcreate_name	string	
# for internal use only
d-i	debconf/frontend	string	
# Invalid hostname
d-i	netcfg/invalid_hostname	error	
# locale
d-i	localechooser/help/locale	note	
# for internal use; can be preseeded
lilo-installer	lilo-installer/skip	boolean	false
# 
# Choices: Antigua and Barbuda, Australia, Botswana, Canada, Hong Kong, India, Ireland, Israel, New Zealand, Nigeria, Philippines, Seychelles, Singapore, South Africa, United Kingdom, United States, Zambia, Zimbabwe, other
d-i	localechooser/shortlist/en	select	
# Write a new empty partition table?
partman-partitioning	partman-partitioning/confirm_write_new_label	boolean	false
# Cannot access repository
apt-setup-udeb	apt-setup/service-failed	error	
# Install the GRUB boot loader to the master boot record?
grub-installer	grub-installer/with_other_os	boolean	true
# No physical volumes selected
partman-lvm	partman-lvm/vgcreate_nosel	error	
# Active devices for the RAID array:
# Choices: 
partman-md	partman-md/raiddevs	multiselect	
# Go back to the menu and correct this problem?
partman-basicfilesystems	partman-basicfilesystems/boot_not_first_partition	boolean	
# Write the changes to disks?
partman-base	partman/confirm	boolean	false
# Passphrase input error
partman-crypto	partman-crypto/passphrase-mismatch	error	
# EFI partition too small
partman-efi	partman-efi/too_small_efi	error	
# Use unrecommended JFS /boot file system?
partman-jfs	partman-jfs/jfs_boot	boolean	false
# Error while deleting volume group
partman-lvm	partman-lvm/vgdelete_error	error	
# iSCSI initiator username for :
partman-iscsi	partman-iscsi/login/username	string	
# Devices to encrypt:
# Choices: 
partman-crypto	partman-crypto/create/partitions	multiselect	
# No physical volume defined in volume group
partman-auto-lvm	partman-auto-lvm/no_pv_in_vg	error	
# Encryption method for this partition:
# Choices: 
partman-crypto	partman-crypto/crypto_type	select	
# No logical volume found
partman-lvm	partman-lvm/lvdelete_nolv	error	
# Unexpected error while creating volume group
partman-auto-lvm	partman-auto-lvm/vg_create_error	error	
# Setting firmware variables for automatic boot
nobootloader	nobootloader/confirmation_powerpc_chrp_pegasos	note	
# Device in use
partman-base	partman-base/devicelocked	error	
# 
# Choices: Bangladesh, India, other
d-i	localechooser/shortlist/bn	select	
# Encryption configuration failure
partman-crypto	partman-crypto/crypto_boot_not_possible	error	
# Failed to run preseeded command
d-i	preseed/command_failed	error	
# for internal use; can be preseeded (deprecated)
d-i	netcfg/disable_dhcp	boolean	false
# Failed to load installer component
d-i	anna/install_failed	error	
# Partitioning method:
# Choices: Guided - use entire disk, Guided - use entire disk and set up LVM, Guided - use entire disk and set up encrypted LVM, Manual
partman-auto	partman-auto/init_automatically_partition	select	50some_device__________regular
# Is the system clock set to UTC?
clock-setup	clock-setup/utc	boolean	true
# Use weak passphrase?
partman-crypto	partman-crypto/weak_passphrase	boolean	false
# Number of spare devices for the RAID array:
partman-md	partman-md/raidsparecount	string	
# Keymap to use:
# Choices: American English, Albanian, Arabic, Asturian, Bangladesh, Belarusian, Bengali, Belgian, Bosnian, Brazilian, British English, Bulgarian (BDS layout), Bulgarian (phonetic layout), Burmese, Canadian French, Canadian Multilingual, Catalan, Chinese, Croatian, Czech, Danish, Dutch, Dvorak, Dzongkha, Esperanto, Estonian, Ethiopian, Finnish, French, Georgian, German, Greek, Gujarati, Gurmukhi, Hebrew, Hindi, Hungarian, Icelandic, Irish, Italian, Japanese, Kannada, Kazakh, Khmer, Kirghiz, Korean, Kurdish (F layout), Kurdish (Q layout), Lao, Latin American, Latvian, Lithuanian, Macedonian, Malayalam, Nepali, Northern Sami, Norwegian, Persian, Philippines, Polish, Portuguese, Punjabi, Romanian, Russian, Serbian (Cyrillic), Sindhi, Sinhala, Slovak, Slovenian, Spanish, Swedish, Swiss French, Swiss German, Tajik, Tamil, Telugu, Thai, Tibetan, Turkish (F layout), Turkish (Q layout), Ukrainian, Uyghur, Vietnamese
d-i	keyboard-configuration/xkb-keymap	select	es
# Install the GRUB boot loader to the master boot record?
grub-installer	grub-installer/only_debian	boolean	true
# Checksum error
d-i	preseed/checksum_error	error	
# Are you sure you want to use a random key?
partman-crypto	partman-crypto/use_random_for_nonswap	boolean	false
# No DHCP client found
d-i	netcfg/no_dhcp_client	error	
# LILO installation target:
# Choices: : Master Boot Record, : new Debian partition, Other choice (Advanced)
lilo-installer	lilo-installer/bootdev	select	
# Domain name:
d-i	netcfg/get_domain	string	
# city
# Choices: Kinshasa, Lubumbashi
tzsetup-udeb	tzsetup/country/CD	select	Africa/Kinshasa
# Are you sure you want to exit now?
d-i	di-utils-reboot/really_reboot	boolean	false
# Devices to remove from the volume group:
# Choices: 
partman-lvm	partman-lvm/vgreduce_parts	multiselect	
# Choose software to install:
# Choices: Debian desktop environment, ... GNOME, ... Xfce, ... KDE Plasma, ... Cinnamon, ... MATE, ... LXDE, ... LXQt, web server, print server, SSH server, standard system utilities
d-i	tasksel/first	multiselect	SSH server, standard system utilities
# 
# Choices: Antarctica
d-i	localechooser/countrylist/Antarctica	select	
# Modules to load:
# Choices: usb-storage (USB storage)
d-i	hw-detect/select_modules	multiselect	usb-storage (USB storage)
# Go back and try a different mirror?
d-i	mirror/no-default	boolean	true
# 
# Choices: Belgium, Germany, Italy, Liechtenstein, Luxembourg, Switzerland, Austria, other
d-i	localechooser/shortlist/de	select	
# Initialization vector generation algorithm for this partition:
# Choices: 
partman-crypto	partman-crypto/ivalgorithm	select	
# Installation complete
finish-install	finish-install/reboot_in_progress	note	
# for internal use; can be preseeded
d-i	preseed/interactive	boolean	false
# Malformed IP address
d-i	netcfg/bad_ipaddress	error	
# Identical labels for two file systems
partman-target	partman-target/same_label	error	
# Debian archive mirror country:
# Choices: enter information manually, Argentina, Armenia, Australia, Austria, Belarus, Belgium, Brazil, Bulgaria, Canada, Chile, China, Costa Rica, Croatia, Czechia, Denmark, El Salvador, Estonia, Finland, France, Georgia, Germany, Greece, Hong Kong, Hungary, India, Indonesia, Iran\, Islamic Republic of, Israel, Italy, Japan, Kazakhstan, Kenya, Korea\, Republic of, Kyrgyzstan, Latvia, Lithuania, Luxembourg, Macedonia\, Republic of, Mexico, Moldova, Netherlands, New Caledonia, New Zealand, Norway, Philippines, Poland, Portugal, Romania, Russian Federation, Réunion, Serbia, Singapore, Slovakia, Slovenia, South Africa, Spain, Sweden, Switzerland, Taiwan, Thailand, Turkey, Ukraine, United Kingdom, United States, Uruguay, Vietnam
d-i	mirror/http/countries	select	ES
# for internal use; can be preseeded
d-i	netcfg/dhcp_timeout	string	25
# Failed to partition the selected disk
partman-auto	partman-auto/no_recipe	error	
# Wireless ESSID for :
d-i	netcfg/wireless_essid_again	string	
# Unable to install 
bootstrap-base	base-installer/kernel/failed-package-install	error	
# zone
# Choices: Port Moresby, Bougainville
tzsetup-udeb	tzsetup/country/PG	select	
# for internal use; can be preseeded
finish-install	finish-install/keep-consoles	boolean	false
# Logical Volume Management
partman-lvm	partman-lvm/help	note	
# Logical volume:
# Choices: 
partman-lvm	partman-lvm/lvdelete_lvnames	select	
# Wireless ESSID for :
d-i	netcfg/wireless_essid	string	
# Unsafe swap space detected
partman-crypto	partman-crypto/unsafe_swap	error	
# Flags for the new partition:
# Choices: 
partman-partitioning	partman-partitioning/set_flags	multiselect	
# What to do with this device:
# Choices: 
partman-base	partman/storage_device	select	
# Return to the menu to set the bootable flag?
partman-ext3	partman-ext3/boot_not_bootable	boolean	
# for internal use; can be preseeded
partman-base	partman/default_filesystem	string	ext4
# Language selection no longer possible
d-i	localechooser/translation/no-select	note	
# Typical usage of this partition:
# Choices: 
partman-basicfilesystems	partman-basicfilesystems/specify_usage	select	
# for internal use; can be preseeded
# Choices: cylinder, minimal, optimal
partman-base	partman/alignment	select	optimal
# Volume group name already in use
partman-auto-lvm	partman-auto-lvm/vg_exists	error	
# Write the changes to the storage devices and configure RAID?
partman-md	partman-md/confirm	boolean	false
# location
# Choices: Guayaquil, Galapagos
tzsetup-udeb	tzsetup/country/EC	select	America/Guayaquil
# Error while running ''
d-i	hw-detect/modprobe_error	error	
# Really delete this software RAID device?
partman-md	partman-md/deleteverify	boolean	false
# Error while creating volume group
partman-lvm	partman-lvm/vgcreate_error	error	
# for internal use; can be preseeded
# Choices: none, safe-upgrade, full-upgrade
pkgsel	pkgsel/upgrade	select	safe-upgrade
# How should the debug logs be saved or transferred?
# Choices: floppy, web, mounted file system
d-i	save-logs/menu	select	
# for internal use; can be preseeded
disk-detect	disk-detect/dmraid/enable	boolean	false
# 
# Choices: Argentina, Bolivia, Brazil, Chile, Colombia, Ecuador, French Guiana, Guyana, Paraguay, Peru, Suriname, Uruguay, Venezuela
d-i	localechooser/countrylist/South_America	select	
# city
# Choices: Ulaanbaatar, Hovd, Choibalsan
tzsetup-udeb	tzsetup/country/MN	select	Asia/Ulaanbaatar
# Participate in the package usage survey?
d-i	popularity-contest/participate	boolean	false
# Keep default keyboard options ()?
d-i	keyboard-configuration/unsupported_options	boolean	true
# Error while creating a new logical volume
partman-lvm	partman-lvm/lvcreate_exists	error	
# No volume group name entered
partman-lvm	partman-lvm/vgcreate_nonamegiven	error	
# How to use this partition:
# Choices: 
partman-target	partman-target/choose_method	select	
# Really erase the data on ?
partman-crypto	partman-crypto/crypto_warn_erase	boolean	false
# Software RAID device to be deleted:
# Choices: 
partman-md	partman-md/deletemenu	select	
# 
# Choices: Brazil, Portugal, other
d-i	localechooser/shortlist/pt	select	
# Failed to install the base system
bootstrap-base	base-installer/debootstrap-failed	error	
# for internal use only
d-i	debian-installer/consoledisplay	string	
# Type of encryption key hash for this partition:
# Choices: 
partman-crypto	partman-crypto/keyhash	select	
# Remove existing logical volume data?
partman-lvm	partman-lvm/device_remove_lvm	boolean	false
# Dummy template for preseeding unavailable questions
d-i	debian-installer/dummy	string	
# Country to base default locale settings on:
# Choices: 
d-i	localechooser/preferred-locale	select	
# DHCP hostname:
d-i	netcfg/dhcp_hostname	string	
# Partition settings:
# Choices: 
partman-base	partman/active_partition	select	
# Web server started
d-i	save-logs/httpd_running	note	
# Proceed with installation to unclean target?
base-installer	base-installer/use_unclean_target	boolean	true
# Downloading a file failed:
# Choices: Retry, Change mirror, Cancel
d-i	retriever/net/error	select	Retry
# Directory in which to save debug logs:
d-i	save-logs/directory	string	/mnt
# Debian archive mirror directory:
d-i	mirror/http/directory	string	/debian/
# Erasing data on  failed
partman-crypto	partman-crypto/crypto_erase_failed	error	
# Choose the next step in the install process:
# Choices: Choose language, Access software for a blind person using a braille display, Configure the keyboard, Detect network hardware, Configure the network, Choose a mirror of the Debian archive, Download installer components, Set up users and passwords, Free memory (low memory install), Configure the clock, Detect disks, Partition disks, Install the base system, Configure the package manager, Select and install software, Install the GRUB boot loader on a hard disk, Install the LILO boot loader on a hard disk, Continue without boot loader, Finish the installation, Change debconf priority, Save debug logs, Execute a shell, Abort the installation
d-i	debian-installer/main-menu	select	Finish the installation
# for internal use; can be preseeded
d-i	debian-installer/exit/halt	boolean	false
# Wait another 30 seconds for hwclock to set the clock?
clock-setup	clock-setup/hwclock-wait	boolean	false
# Write previous changes to disk and continue?
partman-partitioning	partman-partitioning/confirm_resize	boolean	
# Password input error
user-setup-udeb	user-setup/password-mismatch	error	
# for internal use; can be preseeded
d-i	hw-detect/load-ide	boolean	false
# The resize operation is impossible
partman-partitioning	partman-partitioning/impossible_resize	error	
# Invalid input
partman-auto-lvm	partman-auto-lvm/bad_guided_size	error	
# Is this information correct?
d-i	netcfg/confirm_static	boolean	true
# Use a network mirror?
apt-mirror-setup	apt-setup/use_mirror	boolean	
# Use unrecommended JFS root file system?
partman-jfs	partman-jfs/jfs_root	boolean	false
# Empty password
partman-iscsi	partman-iscsi/login/empty_password	error	
# Method for toggling between national and Latin mode:
# Choices: Caps Lock, Right Alt (AltGr), Right Control, Right Shift, Right Logo key, Menu key, Alt+Shift, Control+Shift, Control+Alt, Alt+Caps Lock, Left Control+Left Shift, Left Alt, Left Control, Left Shift, Left Logo key, Scroll Lock key, No toggling
d-i	keyboard-configuration/toggle	select	No toggling
# Continue with the installation?
partman-base	partman/confirm_nochanges	boolean	false
# No volume group found
partman-lvm	partman-lvm/vgreduce_novg	error	
# for internal use; can be preseeded
bootstrap-base	base-installer/excludes	string	
# Encryption package installation failure
partman-crypto	partman-crypto/module_package_missing	error	
# Continue without installing a kernel?
bootstrap-base	base-installer/kernel/skip-install	boolean	false
# Would you like to make this partition active?
lilo-installer	lilo-installer/activate-part	boolean	true
# No partitions to encrypt
partman-crypto	partman-crypto/nothing_to_setup	note	
# Continent or region:
# Choices: Africa, Antarctica, Asia, Atlantic Ocean, Caribbean, Central America, Europe, Indian Ocean, North America, Oceania, South America, other
d-i	localechooser/continentlist	select	Europe
# Continue with partitioning?
partman-partitioning	partman-partitioning/unsupported_label	boolean	false
# Failed to partition the selected disk
partman-auto	partman-auto/autopartitioning_failed	error	
# NTP server to use:
clock-setup	clock-setup/ntp-server	string	0.debian.pool.ntp.org
# Driver needed by your Ethernet card:
# Choices: no ethernet card, , none of the above
d-i	ethdetect/module_select	select	no ethernet card
# Label for the file system in this partition:
partman-basicfilesystems	partman-basicfilesystems/choose_label	string	
# city
# Choices: Western (Sumatra\, Jakarta\, Java\, West and Central Kalimantan), Central (Sulawesi\, Bali\, Nusa Tenggara\, East and South Kalimantan), Eastern (Maluku\, Papua)
tzsetup-udeb	tzsetup/country/ID	select	Asia/Jakarta
# Wireless network:
# Choices:  Enter ESSID manually
d-i	netcfg/wireless_show_essids	select	
# WEP key for wireless device :
d-i	netcfg/wireless_wep	string	
# for internal use; can be preseeded
d-i	preseed/late_command	string	
# 
# Choices: Afghanistan, Bahrain, Bangladesh, Bhutan, Brunei Darussalam, Cambodia, China, Hong Kong, India, Indonesia, Iran\, Islamic Republic of, Iraq, Israel, Japan, Jordan, Kazakhstan, Korea\, Democratic People's Republic of, Korea\, Republic of, Kuwait, Kyrgyzstan, Lao People's Democratic Republic, Lebanon, Macao, Malaysia, Mongolia, Myanmar, Nepal, Oman, Pakistan, Palestine\, State of, Philippines, Qatar, Saudi Arabia, Singapore, Sri Lanka, Syrian Arab Republic, Taiwan, Tajikistan, Thailand, Timor-Leste, Turkey, Turkmenistan, United Arab Emirates, Uzbekistan, Vietnam, Yemen
d-i	localechooser/countrylist/Asia	select	
# for internal use; can be preseeded
d-i	debian-installer/allow_unauthenticated	boolean	false
# location
# Choices: Yap, Truk, Pohnpei, Kosrae
tzsetup-udeb	tzsetup/country/FM	select	Pacific/Ponape
# Kernel to install:
# Choices: linux-image-4.19.0-10-amd64,linux-image-4.19.0-10-amd64-unsigned,linux-image-4.19.0-10-cloud-amd64,linux-image-4.19.0-10-cloud-amd64-unsigned,linux-image-4.19.0-10-rt-amd64,linux-image-4.19.0-10-rt-amd64-unsigned,linux-image-4.19.0-11-amd64,linux-image-4.19.0-11-amd64-unsigned,linux-image-4.19.0-11-cloud-amd64,linux-image-4.19.0-11-cloud-amd64-unsigned,linux-image-4.19.0-11-rt-amd64,linux-image-4.19.0-11-rt-amd64-unsigned,linux-image-amd64,linux-image-amd64-signed-template,linux-image-cloud-amd64,linux-image-rt-amd64, none
bootstrap-base	base-installer/kernel/image	select	linux-image-amd64
# location
# Choices: Madrid, Ceuta, Canary Islands
tzsetup-udeb	tzsetup/country/ES	select	Europe/Madrid
# 
# Choices: Canada, Mexico, Saint Pierre and Miquelon, United States
d-i	localechooser/countrylist/North_America	select	
# Initialisation of encrypted volume failed
partman-crypto	partman-crypto/init_failed	error	
# Keep current keyboard options in the configuration file?
d-i	keyboard-configuration/unsupported_config_options	boolean	true
# Unable to configure GRUB
grub-installer	grub-installer/update-grub-failed	error	
# Unsupported initrd generator
bootstrap-base	base-installer/initramfs/unsupported	error	
# Debootstrap Error
bootstrap-base	base-installer/debootstrap/error/missingrelentry	error	
# Failed to retrieve the preconfiguration file
d-i	preseed/retrieve_error	error	
# Additional locales:
# Choices: aa_DJ.UTF-8, aa_DJ, aa_ER, aa_ER@saaho, aa_ET, af_ZA.UTF-8, af_ZA, agr_PE, ak_GH, am_ET, an_ES.UTF-8, an_ES, anp_IN, ar_AE.UTF-8, ar_AE, ar_BH.UTF-8, ar_BH, ar_DZ.UTF-8, ar_DZ, ar_EG.UTF-8, ar_EG, ar_IN, ar_IQ.UTF-8, ar_IQ, ar_JO.UTF-8, ar_JO, ar_KW.UTF-8, ar_KW, ar_LB.UTF-8, ar_LB, ar_LY.UTF-8, ar_LY, ar_MA.UTF-8, ar_MA, ar_OM.UTF-8, ar_OM, ar_QA.UTF-8, ar_QA, ar_SA.UTF-8, ar_SA, ar_SD.UTF-8, ar_SD, ar_SS, ar_SY.UTF-8, ar_SY, ar_TN.UTF-8, ar_TN, ar_YE.UTF-8, ar_YE, ayc_PE, az_AZ, az_IR, as_IN, ast_ES.UTF-8, ast_ES, be_BY.UTF-8, be_BY, be_BY@latin, bem_ZM, ber_DZ, ber_MA, bg_BG.UTF-8, bg_BG, bhb_IN.UTF-8, bho_IN, bho_NP, bi_VU, bn_BD, bn_IN, bo_CN, bo_IN, br_FR.UTF-8, br_FR, br_FR@euro, brx_IN, bs_BA.UTF-8, bs_BA, byn_ER, ca_AD.UTF-8, ca_AD, ca_ES.UTF-8, ca_ES, ca_ES@euro, ca_ES@valencia, ca_FR.UTF-8, ca_FR, ca_IT.UTF-8, ca_IT, ce_RU, chr_US, cmn_TW, crh_UA, cs_CZ.UTF-8, cs_CZ, csb_PL, cv_RU, cy_GB.UTF-8, cy_GB, da_DK.UTF-8, da_DK, de_AT.UTF-8, de_AT, de_AT@euro, de_BE.UTF-8, de_BE, de_BE@euro, de_CH.UTF-8, de_CH, de_DE.UTF-8, de_DE, de_DE@euro, de_IT.UTF-8, de_IT, de_LI.UTF-8, de_LU.UTF-8, de_LU, de_LU@euro, doi_IN, dsb_DE, dv_MV, dz_BT, el_GR.UTF-8, el_GR, el_GR@euro, el_CY.UTF-8, el_CY, en_AG, en_AU.UTF-8, en_AU, en_BW.UTF-8, en_BW, en_CA.UTF-8, en_CA, en_DK.UTF-8, en_DK.ISO-8859-15, en_DK, en_GB.UTF-8, en_GB, en_GB.ISO-8859-15, en_HK.UTF-8, en_HK, en_IE.UTF-8, en_IE, en_IE@euro, en_IL, en_IN, en_NG, en_NZ.UTF-8, en_NZ, en_PH.UTF-8, en_PH, en_SC.UTF-8, en_SG.UTF-8, en_SG, en_US.UTF-8, en_US, en_US.ISO-8859-15, en_ZA.UTF-8, en_ZA, en_ZM, en_ZW.UTF-8, en_ZW, eo, es_AR.UTF-8, es_AR, es_BO.UTF-8, es_BO, es_CL.UTF-8, es_CL, es_CO.UTF-8, es_CO, es_CR.UTF-8, es_CR, es_CU, es_DO.UTF-8, es_DO, es_EC.UTF-8, es_EC, es_ES.UTF-8, es_ES, es_ES@euro, es_GT.UTF-8, es_GT, es_HN.UTF-8, es_HN, es_MX.UTF-8, es_MX, es_NI.UTF-8, es_NI, es_PA.UTF-8, es_PA, es_PE.UTF-8, es_PE, es_PR.UTF-8, es_PR, es_PY.UTF-8, es_PY, es_SV.UTF-8, es_SV, es_US.UTF-8, es_US, es_UY.UTF-8, es_UY, es_VE.UTF-8, es_VE, et_EE.UTF-8, et_EE, et_EE.ISO-8859-15, eu_ES.UTF-8, eu_ES, eu_ES@euro, eu_FR.UTF-8, eu_FR, eu_FR@euro, fa_IR, ff_SN, fi_FI.UTF-8, fi_FI, fi_FI@euro, fil_PH, fo_FO.UTF-8, fo_FO, fr_BE.UTF-8, fr_BE, fr_BE@euro, fr_CA.UTF-8, fr_CA, fr_CH.UTF-8, fr_CH, fr_FR.UTF-8, fr_FR, fr_FR@euro, fr_LU.UTF-8, fr_LU, fr_LU@euro, fur_IT, fy_NL, fy_DE, ga_IE.UTF-8, ga_IE, ga_IE@euro, gd_GB.UTF-8, gd_GB, gez_ER, gez_ER@abegede, gez_ET, gez_ET@abegede, gl_ES.UTF-8, gl_ES, gl_ES@euro, gu_IN, gv_GB.UTF-8, gv_GB, ha_NG, hak_TW, he_IL.UTF-8, he_IL, hi_IN, hif_FJ, hne_IN, hr_HR.UTF-8, hr_HR, hsb_DE.UTF-8, hsb_DE, ht_HT, hu_HU.UTF-8, hu_HU, hy_AM, hy_AM.ARMSCII-8, ia_FR, id_ID.UTF-8, id_ID, ig_NG, ik_CA, is_IS.UTF-8, is_IS, it_CH.UTF-8, it_CH, it_IT.UTF-8, it_IT, it_IT@euro, iu_CA, ja_JP.UTF-8, ja_JP.EUC-JP, ka_GE.UTF-8, ka_GE, kab_DZ, kk_KZ.UTF-8, kk_KZ, kk_KZ.RK1048, kl_GL.UTF-8, kl_GL, km_KH, kn_IN, ko_KR.UTF-8, ko_KR.EUC-KR, kok_IN, ks_IN, ks_IN@devanagari, ku_TR.UTF-8, ku_TR, kw_GB.UTF-8, kw_GB, ky_KG, lb_LU, lg_UG.UTF-8, lg_UG, li_BE, li_NL, lij_IT, ln_CD, lo_LA, lt_LT.UTF-8, lt_LT, lv_LV.UTF-8, lv_LV, lzh_TW, mag_IN, mai_IN, mai_NP, mfe_MU, mg_MG.UTF-8, mg_MG, mhr_RU, mi_NZ.UTF-8, mi_NZ, miq_NI, mjw_IN, mk_MK.UTF-8, mk_MK, ml_IN, mn_MN, mni_IN, mr_IN, ms_MY.UTF-8, ms_MY, mt_MT.UTF-8, mt_MT, my_MM, nan_TW, nan_TW@latin, nb_NO.UTF-8, nb_NO, nds_DE, nds_NL, ne_NP, nhn_MX, niu_NU, niu_NZ, nl_AW, nl_BE.UTF-8, nl_BE, nl_BE@euro, nl_NL.UTF-8, nl_NL, nl_NL@euro, nn_NO.UTF-8, nn_NO, nr_ZA, nso_ZA, oc_FR.UTF-8, oc_FR, om_ET, om_KE.UTF-8, om_KE, or_IN, os_RU, pa_IN, pa_PK, pap_AW, pap_CW, pl_PL.UTF-8, pl_PL, ps_AF, pt_BR.UTF-8, pt_BR, pt_PT.UTF-8, pt_PT, pt_PT@euro, quz_PE, raj_IN, ro_RO.UTF-8, ro_RO, ru_RU.UTF-8, ru_RU.KOI8-R, ru_RU, ru_RU.CP1251, ru_UA.UTF-8, ru_UA, rw_RW, sa_IN, sah_RU, sat_IN, sc_IT, sd_IN, sd_IN@devanagari, se_NO, sgs_LT, shn_MM, shs_CA, si_LK, sid_ET, sk_SK.UTF-8, sk_SK, sl_SI.UTF-8, sl_SI, sm_WS, so_DJ.UTF-8, so_DJ, so_ET, so_KE.UTF-8, so_KE, so_SO.UTF-8, so_SO, sq_AL.UTF-8, sq_AL, sq_MK, sr_ME, sr_RS, sr_RS@latin, ss_ZA, st_ZA.UTF-8, st_ZA, sv_FI.UTF-8, sv_FI, sv_FI@euro, sv_SE.UTF-8, sv_SE, sv_SE.ISO-8859-15, sw_KE, sw_TZ, szl_PL, ta_IN, ta_LK, tcy_IN.UTF-8, te_IN, tg_TJ.UTF-8, tg_TJ, th_TH.UTF-8, th_TH, the_NP, ti_ER, ti_ET, tig_ER, tk_TM, tl_PH.UTF-8, tl_PH, tn_ZA, to_TO, tpi_PG, tr_CY.UTF-8, tr_CY, tr_TR.UTF-8, tr_TR, ts_ZA, tt_RU, tt_RU@iqtelif, ug_CN, uk_UA.UTF-8, uk_UA, unm_US, ur_IN, ur_PK, uz_UZ.UTF-8, uz_UZ, uz_UZ@cyrillic, ve_ZA, vi_VN, wa_BE.UTF-8, wa_BE, wa_BE@euro, wae_CH, wal_ET, wo_SN, xh_ZA.UTF-8, xh_ZA, yi_US.UTF-8, yi_US, yo_NG, yue_HK, yuw_PG, zh_CN.UTF-8, zh_CN.GB18030, zh_CN.GBK, zh_CN, zh_HK.UTF-8, zh_HK, zh_SG.UTF-8, zh_SG.GBK, zh_SG, zh_TW.UTF-8, zh_TW.EUC-TW, zh_TW, zu_ZA.UTF-8, zu_ZA
d-i	localechooser/supported-locales	multiselect	en_US.UTF-8
# System locale:
# Choices: C, en_US.UTF-8
d-i	debian-installer/locale	select	en_US.UTF-8
# Partition name:
partman-partitioning	partman-partitioning/set_name	string	
# for internal use; can be preseeded
d-i	preseed/include	string	
# Wireless network type for :
# Choices: WEP/Open Network, WPA/WPA2 PSK
d-i	netcfg/wireless_security_type	select	wpa
# Username for your account:
user-setup-udeb	passwd/username	string	pxe-client-user
# Create a normal user account now?
user-setup-udeb	passwd/make-user	boolean	true
# zone
# Choices: Asia/Nicosia (Cyprus (most areas)), Asia/Famagusta (Northern Cyprus)
tzsetup-udeb	tzsetup/country/CY	select	
# for internal use; can be preseeded
d-i	anna/standard_modules	boolean	true
# 
# Choices: Macedonia\, Republic of, Albania, other
d-i	localechooser/shortlist/sq	select	
# Continue installation without /boot partition?
partman-auto-lvm	partman-auto-lvm/no_boot	boolean	
# Type of wireless network:
# Choices: Infrastructure (Managed) network, Ad-hoc network (Peer to peer)
d-i	netcfg/wireless_adhoc_managed	select	Infrastructure (Managed) network
# PCMCIA resource range options:
d-i	hw-detect/pcmcia_resources	string	
# No physical volumes selected
partman-lvm	partman-lvm/vgreduce_nosel	error	
# Remove existing software RAID partitions?
partman-md	partman-md/device_remove_md	boolean	false
# location
# Choices: McMurdo, Rothera, Palmer, Mawson, Davis, Casey, Vostok, Dumont-d'Urville, Syowa
tzsetup-udeb	tzsetup/country/AQ	select	
# Error while initializing physical volume
partman-lvm	partman-lvm/pvcreate_error	error	
# LVM configuration failure
partman-lvm	partman-lvm/commit_failed	error	
# for internal use; can be preseeded
d-i	auto-install/enable	boolean	false
# No usable physical volumes found
partman-lvm	partman-lvm/nopartitions	error	
# Not enough RAID partitions available
partman-md	partman-md/notenoughparts	error	
# for internal use; can be preseeded
partman-auto	partman-auto/expert_recipe_file	string	
# Write the changes to the storage devices and configure RAID?
d-i	partman-md/confirm_nooverwrite	boolean	false
# Volume group to reduce:
# Choices: 
partman-lvm	partman-lvm/vgreduce_names	select	
# 
# Choices: Pakistan, India, other
d-i	localechooser/shortlist/pa	select	
# 
# Choices: Belize, Costa Rica, El Salvador, Guatemala, Honduras, Nicaragua, Panama
d-i	localechooser/countrylist/Central_America	select	
# No logical volume name entered
partman-lvm	partman-lvm/lvcreate_nonamegiven	error	
# Select a location in your time zone:
# Choices: Madrid, Ceuta, Canary Islands, Coordinated Universal Time (UTC)
tzsetup-udeb	time/zone	select	Europe/Madrid
# for internal use; can be preseeded
bootstrap-base	base-installer/debootstrap_script	string	
# Invalid size
partman-partitioning	partman-partitioning/bad_new_partition_size	error	
# state
# Choices: Acre, Alagoas, Amazonas, Amapá, Bahia, Ceará, Distrito Federal, Espírito Santo, Fernando de Noronha, Goiás, Maranhão, Minas Gerais, Mato Grosso do Sul, Mato Grosso, Pará, Paraíba, Pernambuco, Piauí, Paraná, Rio de Janeiro, Rio Grande do Norte, Rondônia, Roraima, Rio Grande do Sul, Santa Catarina, Sergipe, São Paulo, Tocantins
tzsetup-udeb	tzsetup/country/BR	select	America/Sao_Paulo
# for internal use; can be preseeded
# Choices: traditional, label, uuid
partman-target	partman/mount_style	select	uuid
# Force GRUB installation to the EFI removable media path?
grub-installer	grub-installer/force-efi-extra-removable	boolean	true
# 
# Choices: Bouvet Island, Falkland Islands (Malvinas), Saint Helena\, Ascension and Tristan da Cunha, South Georgia and the South Sandwich Islands
d-i	localechooser/countrylist/Atlantic_Ocean	select	
# Not enough RAID partitions specified
partman-auto-raid	partman-auto-raid/notenoughparts	error	
# for internal use; can be preseeded
grub-installer	grub-installer/grub2_instead_of_grub_legacy	boolean	true
# for internal use; can be preseeded
d-i	netcfg/hostname	string	
# for internal use only
bootstrap-base	base-installer/kernel/linux/link_in_boot	boolean	false
# Debootstrap Error
bootstrap-base	base-installer/debootstrap/error/couldntdl	error	
# Type for the new partition:
# Choices: Primary, Logical
partman-partitioning	partman-partitioning/new_partition_type	select	
# state
# Choices: Australian Capital Territory, New South Wales, Victoria, Northern Territory, Queensland, South Australia, Tasmania, Western Australia, Eyre Highway, Yancowinna County, Lord Howe Island
tzsetup-udeb	tzsetup/country/AU	select	Australia/Canberra
# Error while deleting the logical volume
partman-lvm	partman-lvm/lvdelete_error	error	
# Debian archive mirror hostname:
d-i	mirror/ftp/hostname	string	mirror
# Devices to add to the volume group:
# Choices: 
partman-lvm	partman-lvm/vgextend_parts	multiselect	
# Installer components to load:
# Choices: 
d-i	anna/choose_modules_lowmem	multiselect	
# Unable to install GRUB in 
grub-installer	grub-installer/grub-install-failed	error	
# Failed to partition the selected disk
partman-auto-lvm	partman-auto-lvm/unusable_recipe	error	
# Debian archive mirror country:
# Choices: enter information manually
d-i	mirror/https/countries	select	US
# for internal use; can be preseeded
partman-partitioning	partman-partitioning/default_label	string	
# 
partman-base	partman/exception_handler_note	note	
# GRUB installation failed
grub-installer	grub-installer/apt-install-failed	error	
# Not installing to unclean target
base-installer	base-installer/unclean_target_cancel	error	
# for internal use; can be preseeded
partman-auto-raid	partman-auto-raid/recipe	string	
# for internal use; can be preseeded
d-i	preseed/include/checksum	string	
# Start PC card services?
d-i	hw-detect/start_pcmcia	boolean	true
# iSCSI login failed
partman-iscsi	partman-iscsi/login/failed	error	
# 
# Choices: Curaçao
d-i	localechooser/countrylist/other	select	
# IP address:
d-i	netcfg/get_ipaddress	string	
# Keyfile creation failure
partman-crypto	partman-crypto/keyfile-problem	error	
# Continue the install without loading kernel modules?
d-i	anna/no_kernel_modules	boolean	false
# for internal use
d-i	keyboard-configuration/layoutcode	string	es
# Kill switch enabled on 
d-i	netcfg/kill_switch_enabled	note	
# Go back to the menu and correct this problem?
partman-basicfilesystems	partman-basicfilesystems/boot_not_ext2	boolean	
# Architecture not supported
d-i	mirror/noarch	error	
# location
# Choices: Johnston Atoll, Midway Islands, Wake Island
tzsetup-udeb	tzsetup/country/UM	select	Pacific/Midway
# Terminal plugin not available
d-i	debian-installer/terminal-plugin-unavailable	error	
# Required encryption options missing
partman-crypto	partman-crypto/options_missing	error	
# for internal use; can be preseeded
d-i	debian-installer/country	string	ES
# How to use this free space:
# Choices: 
partman-base	partman/free_space	select	
# Do you want to return to the partitioning menu?
partman-basicfilesystems	partman-basicfilesystems/no_mount_point	boolean	
# LVM configuration action:
# Choices: 
partman-lvm	partman-lvm/mainmenu	select	
# Write the changes to disk and configure encrypted volumes?
d-i	partman-crypto/confirm_nooverwrite	boolean	false
# for internal use; can be preseeded
apt-setup-udeb	apt-setup/security_host	string	security.debian.org
# Invalid file system for this mount point
partman-basicfilesystems	partman-basicfilesystems/posix_filesystem_required	error	
# LILO installation failed
lilo-installer	lilo-installer/failed	error	
# Mount point for this partition:
# Choices: / - the root file system, /boot - static files of the boot loader, /home - user home directories, /tmp - temporary files, /usr - static data, /var - variable data, /srv - data for services provided by this system, /opt - add-on application software packages, /usr/local - local hierarchy, Enter manually, Do not mount it
partman-basicfilesystems	partman-basicfilesystems/mountpoint	select	
# for internal use; can be preseeded
d-i	preseed/boot_command	string	
# The size entered is invalid
partman-partitioning	partman-partitioning/bad_new_size	error	
# Partitioning scheme:
# Choices: All files in one partition (recommended for new users), Separate /home partition, Separate /home\, /var\, and /tmp partitions
partman-auto	partman-auto/choose_recipe	select	/lib/partman/recipes/30atomic
# Driver needed for your disk drive:
# Choices: continue with no disk drive, , none of the above
disk-detect	disk-detect/module_select	select	continue with no disk drive
# for internal use; can be preseeded
# Choices: Network Manager, ifupdown (/etc/network/interfaces), No network configuration
d-i	netcfg/target_network_config	select	ifupdown
# for internal use; can be preseeded
partman-auto	partman-auto/method	string	
# Invalid logical volume or volume group name
partman-lvm	partman-lvm/badnamegiven	error	
# Invalid partition name
lilo-installer	lilo-installer/manual_bootdev_error	error	
# for internal use; can be preseeded
d-i	debian-installer/exit/poweroff	boolean	false
# 
# Choices: Andorra, Spain, France, Italy, other
d-i	localechooser/shortlist/ca	select	
# Encryption for this partition:
# Choices: 
partman-crypto	partman-crypto/cipher	select	
# for internal use only
clock-setup	clock-setup/system-time-changed	boolean	true
# for internal use; can be preseeded
partman-auto	partman-auto/disk	string	
# Insufficient memory
d-i	lowmem/insufficient	error	
# Keep the current keyboard layout in the configuration file?
d-i	keyboard-configuration/unsupported_config_layout	boolean	true
# Percentage of the file system blocks reserved for the super-user:
partman-basicfilesystems	partman-basicfilesystems/specify_reserved	string	
# 
# Choices: Brazil, Portugal, other
d-i	localechooser/shortlist/pt_BR	select	
# Go back to the menu and correct errors?
partman-basicfilesystems	partman-basicfilesystems/check_failed	boolean	
# No partitionable media
disk-detect	disk-detect/cannot_find	error	
# Debootstrap Error
bootstrap-base	base-installer/debootstrap/error/nogetrelsig	error	
# Enable source repositories in APT?
apt-setup-udeb	apt-setup/enable-source-repositories	boolean	true
# Invalid mount point
partman-basicfilesystems	partman-basicfilesystems/bad_mountpoint	error	
# Installer components to load:
# Choices: cdrom-checker: Verify the cd contents, cdrom-detect: Detect CDROM devices and mount the CD, crypto-dm-modules-4.19.0-11-amd64-di: devicemapper crypto module, debian-edu-install-udeb: Execute Debian Edu debian-installer profile, debian-edu-profile-udeb: Choose Debian Edu profile, di-utils-exit-installer: Exit installer, driver-injection-disk-detect: Detect OEM driver injection disks, eject-udeb: ejects CDs from d-i menu, emdebian-archive-keyring-udeb: GnuPG keys of the Emdebian archive - udeb, espeakup-udeb: Configure the speech synthesizer voice, event-modules-4.19.0-11-amd64-di: Event support, fuse-modules-4.19.0-11-amd64-di: FUSE modules, iso-scan: Scan hard drives for an installer ISO image, kbd-chooser: Detect a keyboard and select layout, live-installer: Install the system, load-cdrom: Load installer components from CD, load-iso: Load installer components from an installer ISO, load-media: Load installer components from removable media, ltsp-client-builder: build an LTSP environment in the installer target, lvmcfg: Configure the Logical Volume Manager, mbr-udeb: Master Boot Record for IBM-PC compatible computers, mdcfg: Configure MD devices, mouse-modules-4.19.0-11-amd64-di: Mouse support, multipath-modules-4.19.0-11-amd64-di: Multipath support, nbd-modules-4.19.0-11-amd64-di: Network Block Device modules, netcfg-static: Configure a static network, network-console: Continue installation remotely using SSH, openssh-client-udeb: secure shell client for the Debian installer, parted-udeb: Manually partition a hard drive (parted), ppp-modules-4.19.0-11-amd64-di: PPP drivers, ppp-udeb: Point-to-Point Protocol (PPP) - package for Debian Installer, reiserfsprogs-udeb: User-level tools for ReiserFS filesystems, rescue-mode: mount requested partition and start a rescue shell, scsi-nic-modules-4.19.0-11-amd64-di: SCSI drivers for converged NICs, sound-modules-4.19.0-11-amd64-di: sound support, speakup-modules-4.19.0-11-amd64-di: speakup modules, squashfs-modules-4.19.0-11-amd64-di: squashfs modules, udf-modules-4.19.0-11-amd64-di: UDF modules
d-i	anna/choose_modules	multiselect	
# Volume group name overlaps with device name
partman-lvm	partman-lvm/vgcreate_devnameused	error	
# 
# Choices: Argentina, Bolivia, Chile, Colombia, Costa Rica, Cuba, Ecuador, El Salvador, Spain, United States, Guatemala, Honduras, Mexico, Nicaragua, Panama, Paraguay, Peru, Puerto Rico, Dominican Republic, Uruguay, Venezuela, other
d-i	localechooser/shortlist/es	select	
# for internal use; can be preseeded
base-installer	base-installer/install-recommends	boolean	true
# 
# Choices: Cyprus, Turkey, other
d-i	localechooser/shortlist/tr	select	
# Enable shadow passwords?
user-setup-udeb	passwd/shadow	boolean	true
# Drivers to include in the initrd:
# Choices: generic: include all available drivers, targeted: only include drivers needed for this system
bootstrap-base	base-installer/initramfs-tools/driver-policy	select	most
# RAID configuration failure
partman-md	partman-md/commit_failed	error	
# Country of origin for the keyboard:
# Choices: 
d-i	keyboard-configuration/layout	select	
# Installation step failed
d-i	debian-installer/main-menu/item-failure	error	
# for internal use; can be preseeded
d-i	preseed/file/checksum	string	
# Updates management on this system:
# Choices: No automatic updates, Install security updates automatically
pkgsel	pkgsel/update-policy	select	none
# Waiting time (in seconds) for link detection:
d-i	netcfg/link_wait_timeout	string	3
# Debian archive mirror:
# Choices: 
d-i	mirror/https/mirror	select	deb.debian.org
# Unable to automatically remove LVM data
partman-lvm	partman-lvm/device_remove_lvm_span	error	
# Logical Volume Manager not available
partman-lvm	partman-lvm/nolvm	error	
# for internal use; can be preseeded
d-i	debian-installer/add-kernel-opts	string	
# Install the GRUB boot loader to the multipath device?
grub-installer	grub-installer/multipath	boolean	true
# Write the changes to disks?
partman-base	partman/confirm_nooverwrite	boolean	false
# for internal use; can be preseeded
partman-base	partman/early_command	string	
# Compose key:
# Choices: No compose key, Right Alt (AltGr), Right Control, Right Logo key, Menu key, Left Logo key, Caps Lock
d-i	keyboard-configuration/compose	select	No compose key
```
Luego el fichero `/srv/tftp/debian-installer/amd64/boot-screens/syslinux.cfg`, lo modificamos para que quede de la siguiente manera...
```
# D-I config version 2.0
# search path for the c32 support libraries (libcom32, libutil etc.)
path debian-installer/amd64/boot-screens/
include debian-installer/amd64/boot-screens/menu.cfg
default debian-installer/amd64/boot-screens/vesamenu.c32

# Mine
LABEL Install auto
    KERNEL debian-installer/amd64/linux
    INITRD debian-installer/amd64/initrd.gz
    APPEND preseed/url=tftp://192.168.0.2/preseed.cfg auto=true priority=critical
#

prompt 0
timeout 0
```

No he conseguido hacer que cargue el fichero preseed, aunque tftp esté funcionando. Aparece la nueva opción de instalación automática en el menú de debian installer, pero nunca consigue cargar `preseed.cfg`.  









