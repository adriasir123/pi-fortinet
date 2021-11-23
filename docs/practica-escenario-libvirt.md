# Práctica: Instalación y configuración inicial de los servidores Vagrant Libvirt

## Entrega

### Parte 1
> IP "pública" de zeus

`172.22.0.213`

### Parte 2
> Actualizar [esta entrada](https://dit.gonzalonazareno.org/redmine/projects/asir2/wiki/Escenarios_OpenStackKVM_2021-2022) en la wiki de Redmine con información sobre el escenario

Hecho.


## Realización

Escenario de trabajo a usar durante el curso:

![](https://fp.josedomingo.org/hlc2122/u02/img/escenario3.png)

### Ejercicio 1
> Crear escenario Vagrant

Requisitos:
```
vagrant box add rockylinux/8
vagrant box add generic/ubuntu2004
```
*(libvirt)*

`Vagrantfile`:
```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

config.vm.provider :libvirt do |libvirt|
  libvirt.cpus = 1
  libvirt.memory = 512
end

  config.vm.define :zeus do |zeus|
    zeus.vm.box = "debian/bullseye64"
    zeus.vm.hostname = "zeus"
    zeus.vm.network :public_network,
      :dev => "br0",
      :mode => "bridge",
      :type => "bridge"
    zeus.vm.network :private_network,
      :libvirt__network_name => "red-dmz-de-adrianj",
      :libvirt__dhcp_enabled => false,
      :ip => "172.16.0.1",
      :libvirt__netmask => '255.255.0.0',
      :libvirt__forward_mode => "veryisolated"
    zeus.vm.network :private_network,
      :libvirt__network_name => "red-interna-de-adrianj",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.1.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :hera do |hera|
    hera.vm.box = "rockylinux/8"
    hera.vm.hostname = "hera"
    hera.vm.network :private_network,
      :libvirt__network_name => "red-dmz-de-adrianj",
      :libvirt__dhcp_enabled => false,
      :ip => "172.16.0.200",
      :libvirt__netmask => '255.255.0.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :apolo do |apolo|
    apolo.vm.box = "debian/bullseye64"
    apolo.vm.hostname = "apolo"
    apolo.vm.network :private_network,
      :libvirt__network_name => "red-interna-de-adrianj",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.1.102",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :ares do |ares|
    ares.vm.box = "generic/ubuntu2004"
    ares.vm.hostname = "ares"
    ares.vm.network :private_network,
      :libvirt__network_name => "red-interna-de-adrianj",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.1.101",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

### Ejercicio 2
> Actualizar paquetería en todos los servidores

En zeus, apolo y ares:
```
sudo apt update
sudo apt upgrade
```

En hera:
```
sudo dnf check-update
sudo dnf update
```

### Ejercicio 3
> Contraseña en todos los servidores *(para poder modificar desde consola en caso necesario)*

En zeus, apolo, ares y hera:
```
sudo passwd vagrant
```
Contraseña: 1234

### Ejercicio 4
> Crear usuario profesor en todos los servidores. Usa sudo sin contraseña.

#### En zeus, apolo y ares
```
sudo adduser profesor
sudo usermod -aG sudo profesor
```
Contraseña: 1234

`/etc/sudoers`:
```
# Mis cambios
profesor	ALL=(ALL) NOPASSWD:ALL
```
*(al final del fichero)*

#### En hera
```
sudo adduser profesor
sudo passwd profesor
sudo usermod -aG wheel profesor
```
Contraseña: 1234

`/etc/sudoers`:
```
# Mis cambios
profesor	ALL=(ALL) NOPASSWD:ALL
```
*(al final del fichero)*

### Ejercicio 5
> Copiar clave pública ssh de josedom en los servidores al usuario profesor

#### En zeus, apolo y ares

```
mkdir .ssh
touch .ssh/authorized_keys
```

Copio la clave:
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCmjoVIoZCx4QFXvljqozXGqxxlSvO7V2aizqyPgMfGqnyl0J9YXo6zrcWYwyWMnMdRdwYZgHqfiiFCUn2QDm6ZuzC4Lcx0K3ZwO2lgL4XaATykVLneHR1ib6RNroFcClN69cxWsdwQW6dpjpiBDXf8m6/qxVP3EHwUTsP8XaOV7WkcCAqfYAMvpWLISqYme6e+6ZGJUIPkDTxavu5JTagDLwY+py1WB53eoDWsG99gmvyit2O1Eo+jRWN+mgRHIxJTrFtLS6o4iWeshPZ6LvCZ/Pum12Oj4B4bjGSHzrKjHZgTwhVJ/LDq3v71/PP4zaI3gVB9ZalemSxqomgbTlnT jose@debian
```

#### En hera

Lo mismo que arriba, a diferencia de hacer:
```
chmod -R go= ~/.ssh
```

### Ejercicio 6
> Definir nombre largo y corto en todos los servidores, con el dominio `adrianj.gonzalonazareno.org`

#### En zeus

`/etc/hosts`:
```
127.0.0.1	localhost
127.0.0.2	bullseye
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

127.0.1.1 zeus.adrianj.gonzalonazareno.org zeus
```

Compruebo nombre largo:
```
vagrant@zeus:~$ hostname -f
zeus.adrianj.gonzalonazareno.org
```

Compruebo nombre corto:
```
vagrant@zeus:~$ ping zeus -c 1
PING zeus.adrianj.gonzalonazareno.org (127.0.1.1) 56(84) bytes of data.
64 bytes from zeus.adrianj.gonzalonazareno.org (127.0.1.1): icmp_seq=1 ttl=64 time=0.077 ms

--- zeus.adrianj.gonzalonazareno.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.077/0.077/0.077/0.000 ms
```

#### En apolo

`/etc/hosts`:
```
127.0.0.1	localhost
127.0.0.2	bullseye
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

127.0.1.1 apolo.adrianj.gonzalonazareno.org apolo
```

Compruebo nombre largo:
```
vagrant@apolo:~$ hostname -f
apolo.adrianj.gonzalonazareno.org
```

Compruebo nombre corto:
```
vagrant@apolo:~$ ping apolo -c 1
PING apolo.adrianj.gonzalonazareno.org (127.0.1.1) 56(84) bytes of data.
64 bytes from apolo.adrianj.gonzalonazareno.org (127.0.1.1): icmp_seq=1 ttl=64 time=0.070 ms

--- apolo.adrianj.gonzalonazareno.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.070/0.070/0.070/0.000 ms
```

#### En ares

`/etc/hosts`:
```
127.0.0.1	localhost
127.0.1.1	ubuntu2004.localdomain

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

127.0.0.1 ubuntu2004.localdomain

127.0.2.1 ares.adrianj.gonzalonazareno.org ares
```

Compruebo nombre largo:
```
vagrant@ares:~$ hostname -f
ares.adrianj.gonzalonazareno.org
```

Compruebo nombre corto:
```
vagrant@ares:~$ ping ares -c 1
PING ares.adrianj.gonzalonazareno.org (127.0.2.1) 56(84) bytes of data.
64 bytes from ares.adrianj.gonzalonazareno.org (127.0.2.1): icmp_seq=1 ttl=64 time=0.136 ms

--- ares.adrianj.gonzalonazareno.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.136/0.136/0.136/0.000 ms
```

#### En hera

`/etc/hosts`:
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
127.0.1.1 hera.adrianj.gonzalonazareno.org hera
```

Compruebo nombre largo:
```
[vagrant@hera ~]$ hostname -f
hera.adrianj.gonzalonazareno.org
```

Compruebo nombre corto:
```
[vagrant@hera ~]$ ping hera -c 1
PING hera.adrianj.gonzalonazareno.org (127.0.1.1) 56(84) bytes of data.
64 bytes from hera.adrianj.gonzalonazareno.org (127.0.1.1): icmp_seq=1 ttl=64 time=0.071 ms

--- hera.adrianj.gonzalonazareno.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.071/0.071/0.071/0.000 ms
```

### Ejercicio 7
> Instalar bind9 en apolo

```
sudo apt install bind9
```

### Ejercicio 8
> SNAT en zeus + puerta de enlace

```
sudo modprobe iptable_nat
echo iptable_nat >> /etc/modules
echo 1 > /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
sudo apt install iptables-persistent
sudo ip route del default
sudo ip route add default via 172.22.0.1
```

En `/etc/sysctl.conf` descomentar:
```
net.ipv4.ip_forward=1
```

`/etc/network/interfaces`:
```
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
iface eth0 inet dhcp
    post-up ip route del default dev $IFACE
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
auto eth1
iface eth1 inet dhcp
    post-up ip route add default via 172.22.0.1
#VAGRANT-END

#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
auto eth2
iface eth2 inet static
      address 172.16.0.1
      netmask 255.255.0.0
#VAGRANT-END

#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
auto eth3
iface eth3 inet static
      address 10.0.1.1
      netmask 255.255.255.0
#VAGRANT-END
```

> Puerta de enlace apolo

```
sudo ip route del default
sudo ip route add default via 10.0.1.1
```

`/etc/network/interfaces`:
```
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
iface eth0 inet dhcp
    post-up ip route del default dev $IFACE
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
auto eth1
iface eth1 inet static
      address 10.0.1.102
      netmask 255.255.255.0
      gateway 10.0.1.1
#VAGRANT-END
```

> Puerta de enlace ares

```
sudo ip route del default
sudo ip route add default via 10.0.1.1
```

`/etc/netplan/50-vagrant.yaml`:
```
---
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:
      addresses:
          - 10.0.1.101/24
      routes:
          - to: default
            via: 10.0.1.1
```

> Puerta de enlace hera

```
sudo ip route del default
sudo ip route add default via 172.16.0.1
```

`/etc/sysconfig/network-scripts/ifcfg-eth0`:
```
DEVICE="eth0"
BOOTPROTO="dhcp"
DEFROUTE=no
ONBOOT="yes"
TYPE="Ethernet"
PERSISTENT_DHCLIENT="yes"
```

`/etc/sysconfig/network-scripts/ifcfg-eth1`:
```
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
NM_CONTROLLED=yes
BOOTPROTO=none
ONBOOT=yes
IPADDR=172.16.0.200
NETMASK=255.255.0.0
GATEWAY=172.16.0.1
DEFROUTE=yes
DEVICE=eth1
PEERDNS=no
#VAGRANT-END
```

### Ejercicio 9
> ssh config en mi máquina para acceder a todos los servidores

`~/.ssh/config`:
```
#####################
# escenario libvirt #
#####################

Host zeus-clase
    HostName 172.22.0.213
    User vagrant

Host apolo-clase
  HostName 10.0.1.102
  User vagrant
  ProxyJump zeus-clase

Host ares-clase
  HostName 10.0.1.101
  User vagrant
  ProxyJump zeus-clase

Host hera-clase
  HostName 172.16.0.200   
  User vagrant
  ProxyJump zeus-clase
```

En principio esa es mi configuración de conexiones para cuando esté en la red de clase.

Evidentemente, esta configuración crecerá cuando añada conexiones para mi casa.
