# Ejercicio 1: Instalación y configuración DHCP

## ENTREGA

### Parte 1
> Configuración de `isc-dhcp-server`

```
sudo apt update && sudo apt install isc-dhcp-server
```

En `/etc/default/isc-dhcp-server`:
- Descomentar la línea: `DHCPDv4_CONF=/etc/dhcp/dhcpd.conf`
- Especificar por qué interfaz escuchará y servirá IPs: `INTERFACESv4="eth1"`  

En `/etc/dhcp/dhcpd.conf`:
```
subnet 192.168.0.0 netmask 255.255.255.0 {
  range 192.168.0.100 192.168.0.110;
  option subnet-mask 255.255.255.0;
  option routers 192.168.0.1;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  max-lease-time 3600;
}
```

Reiniciar el servicio:
```
sudo systemctl restart isc-dhcp-server
```

### Parte 2
> Mostrar lista de concesiones DHCP

```
sudo dhcp-lease-list --lease
```
```
To get manufacturer names please download http://standards.ieee.org/regauth/oui/oui.txt to /usr/local/etc/oui.txt
Use of uninitialized value $db_cand in -r at /usr/sbin/dhcp-lease-list line 77.
Reading leases from /var/lib/dhcp/dhcpd.leases
MAC                IP              hostname       valid until         manufacturer        
===============================================================================================
52:54:00:ac:7b:64  192.168.0.101   cliente        2021-10-18 18:21:40 -NA-    
```

### Parte 3
> Configurar el cliente por DHCP  

En `/etc/network/interfaces` configurar `eth1` de la siguiente manera:
```
auto eth1
iface eth1 inet dhcp
```

Pedimos configuración DHCP por `eth1`:
```
sudo dhclient eth1
```

### Parte 4
> Hacer un `ip a` en cliente

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:b9:cb:6d brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 192.168.121.68/24 brd 192.168.121.255 scope global dynamic eth0
       valid_lft 3534sec preferred_lft 3534sec
    inet6 fe80::5054:ff:feb9:cb6d/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:ac:7b:64 brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet 192.168.0.101/24 brd 192.168.0.255 scope global dynamic eth1
       valid_lft 3147sec preferred_lft 3147sec
    inet6 fe80::5054:ff:feac:7b64/64 scope link
       valid_lft forever preferred_lft forever
```

### Parte 5
> Servidor DHCP tiene que actuar como router NAT  

```
echo "1" > /proc/sys/net/ipv4/ip_forward
sudo modprobe iptable_nat
sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
```

> Comprobación de resolución DNS en el cliente.

Mediante ping:
```
vagrant@cliente:~$ ping www.example.org
PING www.example.org (93.184.216.34) 56(84) bytes of data.
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=1 ttl=52 time=120 ms
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=2 ttl=52 time=121 ms
^C
--- www.example.org ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 120.205/120.573/120.941/0.368 ms
```

Mediante dig:
```
dig www.example.org
```
```
; <<>> DiG 9.16.15-Debian <<>> www.example.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6677
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.example.org.		IN	A

;; ANSWER SECTION:
www.example.org.	16060	IN	A	93.184.216.34

;; Query time: 24 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Oct 18 19:07:53 UTC 2021
;; MSG SIZE  rcvd: 60
```
Cuando dig se usa sin especificar un DNS, hace la query con nuestro primer nameserver por defecto.  
Utiliza `8.8.8.8`, por lo que es una manera de comprobar si nuestra configuración DNS dada por el DHCP es correcta.  



## Escenario

- Servidor DHCP
- Cliente

Vagrantfile:
```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :servidor do |servidor|
    servidor.vm.box = "debian/bullseye64"
    servidor.vm.hostname = "servidor"
    servidor.vm.network :private_network,
      :libvirt__network_name => "ej1-dhcp",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.1",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :cliente do |cliente|
    cliente.vm.box = "debian/bullseye64"
    cliente.vm.hostname = "cliente"
    cliente.vm.network :private_network,
      :libvirt__network_name => "ej1-dhcp",
      :libvirt__dhcp_enabled => false,
      :libvirt__forward_mode => "veryisolated"
  end

end
```


## Realización

> Comprobar la configuración de red del cliente

```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:ac:7b:64 brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet 192.168.0.101/24 brd 192.168.0.255 scope global dynamic eth1
       valid_lft 2910sec preferred_lft 2910sec
    inet6 fe80::5054:ff:feac:7b64/64 scope link
       valid_lft forever preferred_lft forever
```

Vemos que son correctos los parámetros:
- Rango de IP
- Máscara
- Tiempo de concesión (*ha pasado un rato desde que pedí configuración hasta que hice el copypaste, por eso el lifetime ha bajado*)

```
vagrant@cliente:~$ ip r
default via 192.168.0.1 dev eth1
192.168.0.0/24 dev eth1 proto kernel scope link src 192.168.0.101
192.168.121.0/24 dev eth0 proto kernel scope link src 192.168.121.68
```

Vemos que ha surtido efecto el parámetro `option routers`

```
vagrant@cliente:~$ cat /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Vemos que ha surtido efecto el parámetro `option domain-name-servers`
