# Práctica iptables: perimetral sobre escenario

**ATENCIÓN: La IP pública de Zeus en este escenario es posible que cambie durante las demostraciones ya que este es un escenario que funciona tanto en clase como en casa, y existen 2 direccionamientos distintos.**  



## Parte 1
> Mostrar escenario del que se parte

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

![](https://fp.josedomingo.org/hlc2122/u02/img/escenario3.png)



## Parte 2
> Mostrar el estado iptables del que se parte

Tabla filter:
```
vagrant@zeus:~$ sudo iptables -L -v -n
# Warning: iptables-legacy tables present, use iptables-legacy to see them
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

Tabla nat:
```
vagrant@zeus:~$ sudo iptables -L -v -n -t nat
# Warning: iptables-legacy tables present, use iptables-legacy to see them
Chain PREROUTING (policy ACCEPT 437 packets, 87098 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DNAT       udp  --  eth1   *       0.0.0.0/0            0.0.0.0/0            udp dpt:53 to:10.0.1.102:53
    0     0 DNAT       tcp  --  eth1   *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.16.0.200:80
    0     0 DNAT       tcp  --  eth1   *       0.0.0.0/0            0.0.0.0/0            tcp dpt:25 to:10.0.1.102:25

Chain INPUT (policy ACCEPT 41 packets, 10816 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 12 packets, 873 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain POSTROUTING (policy ACCEPT 4 packets, 257 bytes)
 pkts bytes target     prot opt in     out     source               destination
   36  2986 MASQUERADE  all  --  *      eth1    0.0.0.0/0            0.0.0.0/0
```

Mismas reglas NAT pero en comandos:
```
sudo iptables -t nat -A PREROUTING -i eth1 -p udp --dport 53 -j DNAT --to-destination 10.0.1.102:53
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j DNAT --to-destination 172.16.0.200:80
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 25 -j DNAT --to-destination 10.0.1.102:25
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```



## Parte 3
> Redigir puertos 2222-22 en Zeus desde el exterior

DNAT para mi casa y para clase:
```
sudo iptables -t nat -A PREROUTING -i eth1 -s 172.22.0.0/16 -p tcp --dport 2222 -j DNAT --to-destination 172.22.0.213:22
sudo iptables -t nat -A PREROUTING -i eth1 -s 192.168.1.0/24 -p tcp --dport 2222 -j DNAT --to-destination 192.168.1.115:22
```



## Parte 4
> Permitir tráfico ssh entrante exterior → Zeus

```
sudo iptables -A INPUT -i eth1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -o eth1 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/escenario-libvirt$ ssh zeus-casa
Linux zeus 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

 @@@@@@@@ @@@@@@@@ @@@  @@@  @@@@@@
      @@! @@!      @@!  @@@ !@@
    @!!   @!!!:!   @!@  !@!  !@@!!
  !!:     !!:      !!:  !!!     !:!
 :.::.: : : :: :::  :.:: :  ::.: :

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Feb  6 16:53:51 2022 from 192.168.1.106
vagrant@zeus:~$
```
*(no escribo el puerto 2222 al conectar porque ya está en mi config ssh. Ver la siguiente parte para más información)*

Que funcione esta conexión significa que tanto las reglas de este paso como del anterior funcionan.



## Parte 5
> Permitir tráfico ssh para hacer proxyjump desde zeus

Según como estaba planteado nuestro escenario, se tenía que acceder a la DMZ y a la LAN a través de zeus por ssh. Esto nos obliga a añadir reglas INPUT/OUPUT desde zeus hacia estas redes, para que el salto SSH pueda funcionar.

Muestro la parte correspondiente de mi `.ssh/config`, para que sepamos de dónde partimos:
```
#####################
# escenario libvirt #
#####################

Host zeus-clase
    HostName 172.22.0.213
    User vagrant
    Port 2222

Host zeus-casa
    HostName 192.168.1.115
    User vagrant
    Port 2222


Host apolo-clase
  HostName 10.0.1.102
  User vagrant
  ProxyJump zeus-clase

Host apolo-casa
  HostName 10.0.1.102
  User vagrant
  ProxyJump zeus-casa


Host ares-clase
  HostName 10.0.1.101
  User vagrant
  ProxyJump zeus-clase

Host ares-casa
  HostName 10.0.1.101
  User vagrant
  ProxyJump zeus-casa


Host hera-clase
  HostName 172.16.0.200
  User vagrant
  ProxyJump zeus-clase

Host hera-casa
  HostName 172.16.0.200
  User vagrant
  ProxyJump zeus-casa
```

Tráfico ssh saliente zeus → DMZ:
```
sudo iptables -A OUTPUT -o eth2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i eth2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Tráfico ssh saliente zeus → LAN:
```
sudo iptables -A OUTPUT -o eth3 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i eth3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que puedo conectar a Hera desde mi host:
```
atlas@olympus:~/vagrant/escenario-libvirt$ ssh hera-casa

 @@@  @@@ @@@@@@@@ @@@@@@@   @@@@@@
 @@!  @@@ @@!      @@!  @@@ @@!  @@@
 @!@!@!@! @!!!:!   @!@!!@!  @!@!@!@!
 !!:  !!! !!:      !!: :!!  !!:  !!!
  :   : : : :: :::  :   : :  :   : :

Last login: Sun Feb  6 16:08:56 2022 from 172.16.0.1
[vagrant@hera ~]$
```

Pruebo que puedo conectar a Apolo desde mi host:
```
atlas@olympus:~/vagrant/escenario-libvirt$ ssh apolo-casa
Linux apolo 5.10.0-10-amd64 #1 SMP Debian 5.10.84-1 (2021-12-08) x86_64

  @@@@@@  @@@@@@@   @@@@@@  @@@       @@@@@@
 @@!  @@@ @@!  @@@ @@!  @@@ @@!      @@!  @@@
 @!@!@!@! @!@@!@!  @!@  !@! @!!      @!@  !@!
 !!:  !!! !!:      !!:  !!! !!:      !!:  !!!
  :   : :  :        : :. :  : ::.: :  : :. :

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have mail.
Last login: Sun Feb  6 16:07:35 2022 from 10.0.1.1
vagrant@apolo:~$
```

Pruebo que puedo conectar a Ares desde mi host:
```
atlas@olympus:~/vagrant/escenario-libvirt$ ssh ares-casa

  @@@@@@  @@@@@@@  @@@@@@@@  @@@@@@
 @@!  @@@ @@!  @@@ @@!      !@@
 @!@!@!@! @!@!!@!  @!!!:!    !@@!!
 !!:  !!! !!: :!!  !!:          !:!
  :   : :  :   : : : :: ::: ::.: :

Last login: Thu Feb  3 12:06:44 2022 from 192.168.121.1
vagrant@ares:~$
```



## Parte 6
> Políticas por defecto

```
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP
```

Si no se nos cortan las conexiones ssh a las máquinas después de este paso, las reglas que aplicamos antes funcionan.



## Parte 7
> Permitir tráfico ssh Apolo-Hera → Zeus

```
sudo iptables -A INPUT -s 10.0.1.102,172.16.0.200 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -d 10.0.1.102,172.16.0.200 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona desde Apolo:
```
vagrant@apolo:~$ ssh vagrant@10.0.1.1
Linux zeus 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

 @@@@@@@@ @@@@@@@@ @@@  @@@  @@@@@@
      @@! @@!      @@!  @@@ !@@
    @!!   @!!!:!   @!@  !@!  !@@!!
  !!:     !!:      !!:  !!!     !:!
 :.::.: : : :: :::  :.:: :  ::.: :

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Feb  6 17:38:17 2022 from 10.0.1.102
vagrant@zeus:~$
```

Pruebo que funciona desde Hera:
```
[vagrant@hera ~]$ ssh vagrant@172.16.0.1
Linux zeus 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

 @@@@@@@@ @@@@@@@@ @@@  @@@  @@@@@@
      @@! @@!      @@!  @@@ !@@
    @!!   @!!!:!   @!@  !@!  !@@!!
  !!:     !!:      !!:  !!!     !:!
 :.::.: : : :: :::  :.:: :  ::.: :

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Feb  6 17:38:19 2022 from 10.0.1.102
vagrant@zeus:~$
```



## Parte 8
> Permitir tráfico loopback en Zeus

```
sudo iptables -A OUTPUT -o lo -p icmp -j ACCEPT
sudo iptables -A INPUT -i lo -p icmp -j ACCEPT
```

Ya funciona el ping a localhost:
```
vagrant@zeus:~$ ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.064 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.095 ms
^C
--- 127.0.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1020ms
rtt min/avg/max/mdev = 0.064/0.079/0.095/0.015 ms
```



## Parte 9
> Permitir ping entrante DMZ → Zeus

```
sudo iptables -A INPUT -i eth2 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A OUTPUT -o eth2 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona desde Hera:
```
[vagrant@hera ~]$ ping 172.16.0.1
PING 172.16.0.1 (172.16.0.1) 56(84) bytes of data.
64 bytes from 172.16.0.1: icmp_seq=1 ttl=64 time=0.267 ms
64 bytes from 172.16.0.1: icmp_seq=2 ttl=64 time=0.492 ms
^C
--- 172.16.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1020ms
rtt min/avg/max/mdev = 0.267/0.379/0.492/0.114 ms
```



## Parte 10
> Rechazar ping entrante LAN → Zeus

```
sudo iptables -A INPUT -i eth3 -p icmp -m icmp --icmp-type echo-request -j REJECT
```

Pruebo que bloquea el ping desde Ares:
```
vagrant@ares:~$ ping 10.0.1.1
PING 10.0.1.1 (10.0.1.1) 56(84) bytes of data.
^C
--- 10.0.1.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2037ms
```



## Parte 11
> Bloquear ping entrante exterior → Zeus

La política por defecto de INPUT la tenemos a DROP, y tampoco tenemos reglas en este momento que acepten el ping entrante desde el exterior *(eth1)*:
```
Chain INPUT (policy DROP 37915 packets, 1859K bytes)
 pkts bytes target     prot opt in     out     source               destination
19351 1298K ACCEPT     tcp  --  eth1   *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 state NEW,ESTABLISHED
  221 47538 ACCEPT     tcp  --  eth2   *       0.0.0.0/0            0.0.0.0/0            tcp spt:22 state ESTABLISHED
   83 14061 ACCEPT     tcp  --  eth3   *       0.0.0.0/0            0.0.0.0/0            tcp spt:22 state ESTABLISHED
  115 14256 ACCEPT     tcp  --  *      *       10.0.1.102           0.0.0.0/0            tcp dpt:22 state NEW,ESTABLISHED
   66  9222 ACCEPT     tcp  --  *      *       172.16.0.200         0.0.0.0/0            tcp dpt:22 state NEW,ESTABLISHED
    8   672 ACCEPT     icmp --  lo     *       0.0.0.0/0            0.0.0.0/0
    6   504 ACCEPT     icmp --  eth2   *       0.0.0.0/0            0.0.0.0/0            icmptype 8
   11   924 REJECT     icmp --  eth3   *       0.0.0.0/0            0.0.0.0/0            icmptype 8 reject-with icmp-port-unreachable
```
Esto significa que no necesitamos añadir ninguna regla adicional para que se deniegue el ping entrante a Zeus desde el exterior.



## Parte 12
> Permitir ping saliente Zeus → LAN-DMZ

```
sudo iptables -A OUTPUT -d 10.0.1.0/24,172.16.0.0/16 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A INPUT -s 10.0.1.0/24,172.16.0.0/16 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona el ping a Hera:
```
vagrant@zeus:~$ ping 172.16.0.200
PING 172.16.0.200 (172.16.0.200) 56(84) bytes of data.
64 bytes from 172.16.0.200: icmp_seq=1 ttl=64 time=0.675 ms
64 bytes from 172.16.0.200: icmp_seq=2 ttl=64 time=0.584 ms
^C
--- 172.16.0.200 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1030ms
rtt min/avg/max/mdev = 0.584/0.629/0.675/0.045 ms
```

Pruebo que funciona el ping a Ares:
```
vagrant@zeus:~$ ping 10.0.1.101
PING 10.0.1.101 (10.0.1.101) 56(84) bytes of data.
64 bytes from 10.0.1.101: icmp_seq=1 ttl=64 time=0.532 ms
64 bytes from 10.0.1.101: icmp_seq=2 ttl=64 time=0.359 ms
^C
--- 10.0.1.101 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1020ms
rtt min/avg/max/mdev = 0.359/0.445/0.532/0.086 ms
```



## Parte 13
> Permitir ping saliente Zeus → exterior

```
sudo iptables -A OUTPUT -o eth1 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A INPUT -i eth1 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona:
```
vagrant@zeus:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=19.1 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=19.8 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 19.103/19.468/19.834/0.365 ms
```



## Parte 14
> Permitir ping Hera → LAN

```
sudo iptables -A FORWARD -s 172.16.0.200 -o eth3 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A FORWARD -i eth3 -d 172.16.0.200 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona haciendo ping a Ares:
```
[vagrant@hera ~]$ ping 10.0.1.101
PING 10.0.1.101 (10.0.1.101) 56(84) bytes of data.
64 bytes from 10.0.1.101: icmp_seq=1 ttl=63 time=1.94 ms
64 bytes from 10.0.1.101: icmp_seq=2 ttl=63 time=0.898 ms
^C
--- 10.0.1.101 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.898/1.417/1.937/0.520 ms
```



## Parte 15
> Permitir tráfico ssh Hera → LAN

```
sudo iptables -A FORWARD -s 172.16.0.200 -o eth3 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth3 -d 172.16.0.200 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona haciendo ssh a Ares:
```
[vagrant@hera ~]$ ssh vagrant@10.0.1.101

  @@@@@@  @@@@@@@  @@@@@@@@  @@@@@@
 @@!  @@@ @@!  @@@ @@!      !@@
 @!@!@!@! @!@!!@!  @!!!:!    !@@!!
 !!:  !!! !!: :!!  !!:          !:!
  :   : :  :   : : : :: ::: ::.: :

Last login: Mon Feb  7 00:09:02 2022 from 172.16.0.200
vagrant@ares:~$
```



## Parte 16
> Permitir tráfico ssh LAN → Hera

```
sudo iptables -A FORWARD -i eth3 -d 172.16.0.200 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.200 -o eth3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona desde Ares:
```
vagrant@ares:~$ ssh vagrant@172.16.0.200

 @@@  @@@ @@@@@@@@ @@@@@@@   @@@@@@
 @@!  @@@ @@!      @@!  @@@ @@!  @@@
 @!@!@!@! @!!!:!   @!@!!@!  @!@!@!@!
 !!:  !!! !!:      !!: :!!  !!:  !!!
  :   : : : :: :::  :   : :  :   : :

Last login: Sun Feb  6 16:56:24 2022 from 172.16.0.1
[vagrant@hera ~]$
```



## Parte 17
> Permitir ping LAN-DMZ → exterior

```
sudo iptables -A FORWARD -s 10.0.1.0/24,172.16.0.0/16 -o eth1 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A FORWARD -i eth1 -d 10.0.1.0/24,172.16.0.0/16 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Tenemos, porque lo hicimos en una anterior práctica sobre el escenario, una regla de SNAT que entra en juego en este apartado, así que no la tenemos que volver a añadir. Es la siguiente:
```
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```

Pruebo que funciona desde Hera:
```
[vagrant@hera ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=19.4 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=14.7 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 14.739/17.069/19.400/2.334 ms
```

Pruebo que funciona desde Ares:
```
vagrant@ares:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=14.4 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=14.1 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 14.081/14.255/14.430/0.174 ms
```



## Parte 18
> LAN puede navegar

```
sudo iptables -A FORWARD -i eth3 -o eth1 -p tcp -m multiport --dport 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth3 -p tcp -m multiport --sport 80,443 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona el tráfico http desde Ares:
```
vagrant@ares:~$ telnet 52.47.209.216 80
Trying 52.47.209.216...
Connected to 52.47.209.216.
Escape character is '^]'.

```

Pruebo que funciona el tráfico https desde Apolo:
```
vagrant@apolo:~$ telnet 52.47.209.216 443
Trying 52.47.209.216...
Connected to 52.47.209.216.
Escape character is '^]'.

```



## Parte 19
> Hera puede navegar

```
sudo iptables -A FORWARD -s 172.16.0.200 -o eth1 -p tcp -m multiport --dport 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -d 172.16.0.200 -p tcp -m multiport --sport 80,443 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona el tráfico http:
```
[vagrant@hera ~]$ telnet 52.47.209.216 80
Trying 52.47.209.216...
Connected to 52.47.209.216.
Escape character is '^]'.

```

Pruebo que funciona el tráfico https:
```
[vagrant@hera ~]$ telnet 52.47.209.216 443
Trying 52.47.209.216...
Connected to 52.47.209.216.
Escape character is '^]'.

```



## Parte 20

Tenemos que instalar en Hera un servidor de correos y un servidor ftp.

### Servidor de correo

Instalo:
```
sudo dnf install postfix
```

Modifico `/etc/postfix/main.cf` para que acepte peticiones desde cualquier IP y no sólo desde localhost *(es como está por defecto)*:
```
inet_interfaces = all
```

Inicio postfix:
```
sudo systemctl start postfix
```

Compruebo que está escuchando el puerto correctamente:
```
[vagrant@hera ~]$ sudo ss -tulpn | grep LISTEN
tcp   LISTEN 0      128          0.0.0.0:111       0.0.0.0:*    users:(("rpcbind",pid=603,fd=4),("systemd",pid=1,fd=88))
tcp   LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=670,fd=4))
tcp   LISTEN 0      100          0.0.0.0:25        0.0.0.0:*    users:(("master",pid=32765,fd=16))
tcp   LISTEN 0      128             [::]:111          [::]:*    users:(("rpcbind",pid=603,fd=6),("systemd",pid=1,fd=90))
tcp   LISTEN 0      128                *:80              *:*    users:(("httpd",pid=720,fd=4),("httpd",pid=719,fd=4),("httpd",pid=718,fd=4),("httpd",pid=701,fd=4))
tcp   LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=670,fd=6))
tcp   LISTEN 0      100             [::]:25           [::]:*    users:(("master",pid=32765,fd=17))
```

### Servidor FTP

```
sudo dnf install vsftpd
sudo systemctl start vsftpd
```

Compruebo que está escuchando el puerto correctamente:
```
[vagrant@hera ~]$ sudo ss -tulpn | grep LISTEN
tcp   LISTEN 0      128          0.0.0.0:111       0.0.0.0:*    users:(("rpcbind",pid=603,fd=4),("systemd",pid=1,fd=88))
tcp   LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=670,fd=4))
tcp   LISTEN 0      100          0.0.0.0:25        0.0.0.0:*    users:(("master",pid=32765,fd=16))
tcp   LISTEN 0      128             [::]:111          [::]:*    users:(("rpcbind",pid=603,fd=6),("systemd",pid=1,fd=90))
tcp   LISTEN 0      128                *:80              *:*    users:(("httpd",pid=720,fd=4),("httpd",pid=719,fd=4),("httpd",pid=718,fd=4),("httpd",pid=701,fd=4))
tcp   LISTEN 0      32                 *:21              *:*    users:(("vsftpd",pid=32862,fd=3))
tcp   LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=670,fd=6))
tcp   LISTEN 0      100             [::]:25           [::]:*    users:(("master",pid=32765,fd=17))
```



## Parte 21
> Permitir tráfico exterior → Hera para Web-FTP

La regla DNAT web ya la tenía previamente, así que no hace falta añadirla. La recuerdo:
```
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j DNAT --to-destination 172.16.0.200:80
```

Añado la regla DNAT FTP:
```
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 21 -j DNAT --to-destination 172.16.0.200:21
```

Reglas forward web:
```
sudo iptables -A FORWARD -i eth1 -d 172.16.0.200 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.200 -o eth1 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona el tráfico http a Hera desde mi host:
```
atlas@olympus:~$ telnet 172.22.0.213 80
Trying 172.22.0.213...
Connected to 172.22.0.213.
Escape character is '^]'.

```

Reglas forward ftp:
```
sudo iptables -A FORWARD -i eth1 -d 172.16.0.200 -p tcp --dport 21 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.200 -o eth1 -p tcp --sport 21 -j ACCEPT
```

Pruebo que funciona el tráfico ftp a Hera desde mi host:
```
atlas@olympus:~$ telnet 172.22.0.213 21
Trying 172.22.0.213...
Connected to 172.22.0.213.
Escape character is '^]'.
220 (vsFTPd 3.0.3)

```



## Parte 22
> Permitir tráfico LAN → Hera para Web-FTP

Reglas forward web:
```
sudo iptables -A FORWARD -i eth3 -d 172.16.0.200 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.200 -o eth3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona el tráfico http a Hera desde Ares:
```
vagrant@ares:~$ telnet 172.16.0.200 80
Trying 172.16.0.200...
Connected to 172.16.0.200.
Escape character is '^]'.

```

Reglas forward ftp:
```
sudo iptables -A FORWARD -i eth3 -d 172.16.0.200 -p tcp --dport 21 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.200 -o eth3 -p tcp --sport 21 -j ACCEPT
```

Pruebo que funciona el tráfico ftp a Hera desde Apolo:
```
vagrant@apolo:~$ telnet 172.16.0.200 21
Trying 172.16.0.200...
Connected to 172.16.0.200.
Escape character is '^]'.
220 (vsFTPd 3.0.3)

```



## Parte 23
> Permitir tráfico LAN → Hera al servidor correo, pero no se podrá acceder desde otro sitio

```
sudo iptables -A FORWARD -i eth3 -d 172.16.0.200 -p tcp --dport 25 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.200 -o eth3 -p tcp --sport 25 -j ACCEPT
```

Pruebo que funciona desde Ares:
```
vagrant@ares:~$ telnet 172.16.0.200 25
Trying 172.16.0.200...
Connected to 172.16.0.200.
Escape character is '^]'.
220 hera.localdomain ESMTP Postfix

```

Realmente no tendría por qué hacerlo porque no influye, pero para que quede todo más limpio, eliminaré la regla DNAT 25 que teníamos previamente por el escenario:

![](https://i.imgur.com/jv4yvRR.png)

Desde el exterior igualmente no podríamos tener acceso al servidor de correos antiguo de Apolo porque aunque tengamos esa regla DNAT, no hemos hecho en ningún momento reglas forward hacia Apolo.

La elimino:
```
sudo iptables -t nat -D PREROUTING -i eth1 -p tcp --dport 25 -j DNAT --to-destination 10.0.1.102:25
```



## Parte 24
> Permitir tráfico DMZ → Ares al servidor mysql. No se podrá acceder desde el exterior

```
sudo iptables -A FORWARD -i eth2 -d 10.0.1.101 -p tcp --dport 3306 -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.101 -o eth2 -p tcp --sport 3306 -j ACCEPT
```

Pruebo que funciona desde Hera:
```
[vagrant@hera ~]$ sudo mysql -u root -h 10.0.1.101 -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 45
Server version: 10.3.31-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Desde el exterior no podremos acceder claramente, porque no existe una regla DNAT válida:

![](https://i.imgur.com/T3uQOhq.png)



## Parte 25
> Bloquear ICMP Flood limitando el número de peticiones por segundo

Desde el exterior y la LAN hemos bloqueado el tráfico ICMP así que no tenemos que preocuparnos por ataques de flood. **Desde la DMZ sí tenemos permitido el tráfico, así que lo tenemos que limitar**.

Primero voy a realizar un ataque desde Hera a Zeus sin límite de protección y mostraré lo que sucedería.

Este es el comando de ataque que ejecutaré en Hera:
```
sudo ping -f -s 56500 172.16.0.1
```

Hago el ataque, y así se quedarían los contadores después de varios segundos:

![](https://i.imgur.com/onPFJxU.png)

Borro la regla, y la vuelvo a añadir con un límite de protección:
```
sudo iptables -A INPUT -i eth2 -p icmp -m icmp --icmp-type echo-request -m limit --limit 1/s --limit-burst 1 -j ACCEPT
```

Vuelvo a lanzar el ataque, y veo cómo se quedan los contadores después de varios segundos:

![](https://i.imgur.com/sVce9V0.png)

4 paquetes, habiendo limitado el tráfico a 1 por segundo significa que he lanzado el ataque por 4 segundos, pero sólo 1 paquete se permitió gracias al límite. Funciona correctamente.



## Parte 26
> Bloquear SYN Flood

Necesitamos esta regla para limitar:
```
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
```

**PERO** en mi caso no la necesito, porque he comprobado que la política por defecto DROP en INPUT bloquea el ataque de flood.

Este es el comando de ataque que necesitamos, usando como ejemplo el puerto 80 *(partiendo de que sabemos que está abierto y redirigido a Hera)*:
```
sudo hping3 -p 80 --flood --rand-source 192.168.1.115
```

Borro los contadores para esta prueba, y lanzo el ataque durante varios segundos. Vemos que la política DROP ha cortado el ataque:

![](https://i.imgur.com/HvUrZSN.png)

Sabemos que esos paquetes son del ataque porque el contador estaba a 0 previamente, y me fijé que no aumentase al no hacer nada por varios segundos.  
También hay que tener en cuenta que esa cantidad de paquetes en varios segundos no es algo normal, teniendo en cuenta que durante la prueba el único tráfico que estaba llegando a Zeus era el ataque.



## Parte 27

Bloquear escaneos de puertos a Zeus.

### Lógica base
Según he investigado existen [ciertas reglas](https://serverfault.com/questions/941952/iptables-rules-and-port-scanners-blocking) que dicen bloquear los escaneos de puertos, pero al probarlas no funcionan.

Después de indagar he llegado a [este post](https://serverfault.com/questions/246225/how-can-i-use-iptable-rules-to-prevent-port-scanning-like-hping-and-nmap) que dice algo bastante interesante:

_"Lo lógico es usar una política DROP y sólo permitir el tráfico que queramos. No hay manera alguna de bloquear escaneos de puertos sin que se llegue a bloquear tráfico legítimo."_

Esto además de interesante tiene mucho sentido, porque si bloqueáramos tráfico a todos los puertos a su vez perderíamos todo tipo de conexión que fuese legítima. Evidentemente esto no es lo que queremos.

Al final, esto se traduce a que cuando se use nmap se mostrarán los puertos abiertos, pero ninguno más. Queda como tarea del administrador el que el tráfico a esos puertos esté correctamente protegido y dirigido.

### Prueba 1

Partiendo de la lógica base que es casi cierta en su totalidad, realmente hay algunas medidas que podemos tomar.

Intentaremos logear accesos a un puerto que sabemos que está cerrado. Si intentan acceder a él, se puede entender que es un atacante porque alguien que quiera acceder legítimamente sabría qué puertos están abiertos.

Regla:
```
sudo iptables -A INPUT -p tcp -m tcp --dport 19 -j LOG --log-level 1 --log-prefix "PortScan"
```

Lanzamos nmap desde nuestro host a Zeus:
```
atlas@olympus:~$ nmap 172.22.0.213 -Pn
Starting Nmap 7.80 ( https://nmap.org ) at 2022-02-08 12:20 CET
Nmap scan report for 172.22.0.213
Host is up (0.0011s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE
22/tcp   open  ssh
2222/tcp open  EtherNetIP-1

Nmap done: 1 IP address (1 host up) scanned in 8.15 seconds
```

Mostramos lo que ha aparecido en `/var/log/syslog`:
```
Feb  8 11:20:47 zeus kernel: [  453.591450] PortScanIN=eth1 OUT= MAC=52:54:00:6d:6e:a7:06:68:af:97:62:7f:08:00 SRC=172.22.9.227 DST=172.22.0.213 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=63993 DF PROTO=TCP SPT=35550 DPT=19 WINDOW=64240 RES=0x00 SYN URGP=0
```

Esto funciona, e incluso logea la IP de origen del atacante, pero no es una solución.  
Esto simplemente es un log, no estamos bloqueando el ataque nmap y el atacante seguirá teniendo una respuesta de los puertos abiertos.

### Solución 1: Bloquear Null scan

Lanzo este ataque desde mi host a Zeus:
```
atlas@olympus:~$ sudo nmap -sN 172.22.0.213
Starting Nmap 7.80 ( https://nmap.org ) at 2022-02-09 08:48 CET
Stats: 0:00:14 elapsed; 0 hosts completed (1 up), 1 undergoing NULL Scan
NULL Scan Timing: About 66.60% done; ETC: 08:48 (0:00:08 remaining)
Stats: 0:00:21 elapsed; 0 hosts completed (1 up), 1 undergoing NULL Scan
NULL Scan Timing: About 99.99% done; ETC: 08:48 (0:00:00 remaining)
Nmap scan report for 172.22.0.213
Host is up (0.00041s latency).
All 1000 scanned ports on 172.22.0.213 are open|filtered
MAC Address: 52:54:00:6D:6E:A7 (QEMU virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 21.27 seconds
```

Gracias a las políticas por defecto DROP de la tabla filter, este tipo de escaneo está fallando.

Lo último que queda sería logear el ataque y bloquear al atacante por IP:
```
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -m limit --limit 3/m --limit-burst 5 -j LOG --log-prefix "Firewall> Null scan "
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -m recent --name blacklist_60 --set -m comment --comment "Drop/Blacklist Null scan" -j DROP
```

Compruebo `/var/log/syslog`:

![](https://i.imgur.com/SFq55HX.png)

### Solución 2: Bloquear Xmas scan

Lanzo este ataque desde mi host a Zeus:
```
atlas@olympus:~$ sudo nmap -sX 172.22.0.213
Starting Nmap 7.80 ( https://nmap.org ) at 2022-02-09 09:08 CET
Nmap scan report for 172.22.0.213
Host is up (0.00026s latency).
All 1000 scanned ports on 172.22.0.213 are open|filtered
MAC Address: 52:54:00:6D:6E:A7 (QEMU virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 21.25 seconds
```

Gracias a las políticas por defecto DROP de la tabla filter, este tipo de escaneo está fallando, no me aparece ninguna información sobre los puertos abiertos.

Lo último que queda sería logear el ataque y bloquear al atacante por IP:
```
sudo iptables -A INPUT -p tcp --tcp-flags ALL FIN,PSH,URG -m limit --limit 3/m --limit-burst 5 -j LOG --log-prefix "Firewall> XMAS scan "
sudo iptables -A INPUT -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -m limit --limit 3/m --limit-burst 5 -j LOG --log-prefix "Firewall> XMAS-PSH scan "
sudo iptables -A INPUT -p tcp --tcp-flags ALL ALL -m limit --limit 3/m --limit-burst 5 -j LOG --log-prefix "Firewall> XMAS-ALL scan "
sudo iptables -A INPUT -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -m recent --name blacklist_60 --set  -m comment --comment "Drop/Blacklist Xmas/PSH scan" -j DROP
sudo iptables -A INPUT -p tcp --tcp-flags ALL FIN,PSH,URG -m recent --name blacklist_60 --set  -m comment --comment "Drop/Blacklist Xmas scan" -j DROP
sudo iptables -A INPUT -p tcp --tcp-flags ALL ALL -m recent --name blacklist_60 --set  -m comment --comment "Drop/Blacklist Xmas/All scan" -j DROP
```

Compruebo `/var/log/syslog`:

![](https://i.imgur.com/M7zswAp.png)

### Solución 3: Bloquear FIN scan

Lanzo este ataque desde mi host a Zeus:
```
atlas@olympus:~$ sudo nmap -sF 172.22.0.213
Starting Nmap 7.80 ( https://nmap.org ) at 2022-02-09 09:32 CET
Nmap scan report for 172.22.0.213
Host is up (0.00017s latency).
All 1000 scanned ports on 172.22.0.213 are open|filtered
MAC Address: 52:54:00:6D:6E:A7 (QEMU virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 21.20 seconds
```

Gracias a las políticas por defecto DROP de la tabla filter, este tipo de escaneo está fallando, no me aparece ninguna información sobre los puertos abiertos.

Lo último que queda sería logear el ataque y bloquear al atacante por IP:
```
sudo iptables -A INPUT -p tcp --tcp-flags ALL FIN -m limit --limit 3/m --limit-burst 5 -j LOG --log-prefix "Firewall> FIN scan "
sudo iptables -A INPUT -p tcp --tcp-flags ALL FIN -m recent --name blacklist_60 --set  -m comment --comment "Drop/Blacklist FIN scan" -j DROP
```

Compruebo `/var/log/syslog`:

![](https://i.imgur.com/xpKrh6C.png)

### Solución 4: Bloquear UDP scan

Lanzo el scan desde mi host a Zeus:
```
atlas@olympus:~$ sudo nmap -sU 172.22.0.213
Starting Nmap 7.80 ( https://nmap.org ) at 2022-02-09 10:45 CET
Nmap scan report for 172.22.0.213
Host is up (0.00016s latency).
All 1000 scanned ports on 172.22.0.213 are open|filtered
MAC Address: 52:54:00:6D:6E:A7 (QEMU virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 21.19 seconds
atlas@olympus:~$
```

Gracias a las políticas por defecto DROP de la tabla filter, este tipo de escaneo está fallando, no me aparece ninguna información sobre los puertos abiertos.

Lo último que queda sería logear el ataque y bloquear el tráfico específico anormal:
```
sudo iptables -A INPUT -p udp -m limit --limit 6/h --limit-burst 1 -m length --length 0:28 -j LOG --log-prefix "Firewall>0 length udp "
sudo iptables -A INPUT -p udp -m length --length 0:28 -m comment --comment "Drop UDP packet with no content" -j DROP
```

En syslog no me aparece nada porque no tengo tráfico udp permitido.



## Parte 28

Modificar el cortafuegos todo lo necesario para que los servicios antiguos sigan funcionando.

### Tráfico DNS externa → Apolo

```
sudo iptables -A FORWARD -i eth1 -d 10.0.1.102 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.102 -o eth1 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona desde mi host:
```
atlas@olympus:~$ dig NS adrianj.gonzalonazareno.org

; <<>> DiG 9.16.15-Debian <<>> NS adrianj.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1376
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 976da02d0426b93be97b71876204c09c9e25924aa113dc62 (good)
;; QUESTION SECTION:
;adrianj.gonzalonazareno.org.	IN	NS

;; ANSWER SECTION:
adrianj.gonzalonazareno.org. 86229 IN	NS	zeus.adrianj.gonzalonazareno.org.

;; Query time: 0 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Thu Feb 10 08:36:59 CET 2022
;; MSG SIZE  rcvd: 103
```

### Tráfico DNS Apolo → externa

```
sudo iptables -A FORWARD -s 10.0.1.102 -o eth1 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -d 10.0.1.102 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona desde Apolo:
```
vagrant@apolo:~$ dig @127.0.0.1 www.google.es

; <<>> DiG 9.16.22-Debian <<>> @127.0.0.1 www.google.es
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9760
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: fc8dc3dfcf8ed8c7010000006204c5d28d2cd0f4696bab50 (good)
;; QUESTION SECTION:
;www.google.es.	        IN	A

;; ANSWER SECTION:
www.google.es.	        300	IN	A	216.58.215.131

;; Query time: 92 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Feb 10 07:59:14 UTC 2022
;; MSG SIZE  rcvd: 86
```

Pruebo que funciona desde Ares:
```
vagrant@ares:~$ dig fp.josedomingo.org

; <<>> DiG 9.16.1-Ubuntu <<>> fp.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45695
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 653a9b32ddd84ca8010000006204c6b054cab2e10d50bf35 (good)
;; QUESTION SECTION:
;fp.josedomingo.org.	        IN	A

;; ANSWER SECTION:
fp.josedomingo.org.	900	IN	CNAME	endor.josedomingo.org.
endor.josedomingo.org.	900	IN	A	37.187.119.60

;; Query time: 2705 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Feb 10 08:03:32 UTC 2022
;; MSG SIZE  rcvd: 111
```
*(funciona sin reglas adicionales porque Ares y Apolo están en la misma red)*

Para que Zeus pueda hacer consultas DNS usando Apolo, necesitamos reglas adicionales:
```
sudo iptables -A OUTPUT -d 10.0.1.102 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -s 10.0.1.102 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona desde Zeus:
```
vagrant@zeus:~$ dig www.josedomingo.org

; <<>> DiG 9.16.22-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37724
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 1e95cdcc67defb84010000006204c9120b793d41bff868be (good)
;; QUESTION SECTION:
;www.josedomingo.org.	        IN	A

;; ANSWER SECTION:
www.josedomingo.org.	889	IN	CNAME	endor.josedomingo.org.
endor.josedomingo.org.	290	IN	A	37.187.119.60

;; Query time: 0 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Feb 10 08:13:06 UTC 2022
;; MSG SIZE  rcvd: 112
```

Para que desde la DMZ se puedan hacer consultas DNS usando Apolo, necesitamos reglas adicionales:
```
sudo iptables -A FORWARD -i eth2 -d 10.0.1.102 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.102 -o eth2 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona desde Hera:
```
[vagrant@hera ~]$ dig stackoverflow.com

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> stackoverflow.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64139
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: fcf5085ad3e0c20e010000006204cb4d63eb8fac5764ecd5 (good)
;; QUESTION SECTION:
;stackoverflow.com.	        IN	A

;; ANSWER SECTION:
stackoverflow.com.	300	IN	A	151.101.1.69
stackoverflow.com.	300	IN	A	151.101.193.69
stackoverflow.com.	300	IN	A	151.101.129.69
stackoverflow.com.	300	IN	A	151.101.65.69

;; Query time: 1944 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Feb 10 08:22:36 UTC 2022
;; MSG SIZE  rcvd: 138
```

### Tráfico http externa → Hera

La regla DNAT no hace falta añadirla, porque ya la teníamos de antes:

![](https://i.postimg.cc/DZn2s1ZG/dnat-80-web.png)

Las reglas forward no hacen falta añadirlas porque ya las hemos hecho durante esta práctica:

![](https://i.postimg.cc/rmLRZhVY/forward-http-hera.png)

Desde fuera, compruebo que puedo acceder a `www.adrianj.gonzalonazareno.org`:

![](https://i.postimg.cc/4yYrq1Nj/acceso-web-externa-hera-www.png)

### Tráfico correo Apolo → externa

```
sudo iptables -A FORWARD -s 10.0.1.102 -o eth1 -p tcp --dport 25 -j ACCEPT
sudo iptables -A FORWARD -i eth1 -d 10.0.1.102 -p tcp --sport 25 -j ACCEPT
```

Envío un correo a mi gmail desde Apolo:
```
vagrant@apolo:~$ mail -r vagrant@adrianj.gonzalonazareno.org adristudy@gmail.com
Cc:
Subject: hola
hola

```

Muestro que me ha llegado:

![](https://i.imgur.com/BxVGGh3.png)

### Tráfico correo externa → Apolo

Vuelvo a añadir la regla DNAT que previamente eliminé:
```
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 25 -j DNAT --to-destination 10.0.1.102:25
```

Habilito el tráfico de vuelta:
```
sudo iptables -A FORWARD -i eth1 -d 10.0.1.102 -p tcp --dport 25 -j ACCEPT
```

Envío desde gmail un correo a Apolo:

![](https://i.imgur.com/4b87Myc.png)

Compruebo en Apolo que me haya llegado:

![](https://i.imgur.com/JF2VMhe.png)

Muestro el contenido para que se vea que es el correo correcto:

![](https://i.imgur.com/8Qr4uvS.png)



## Parte 29
> El cortafuegos tiene que funcionar después de un reinicio

Necesitamos el paquete `iptables-persistent`, que no tenemos que instalar porque ya se hizo en prácticas sobre el escenario anteriores.

Teniendo el paquete instalado, para guardar las reglas ejecuto este comando:
```
iptables-save > /etc/iptables/rules.v4
```
Se guardarán en ese fichero, y eso las hará persistentes en reinicios.
