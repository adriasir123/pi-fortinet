# Práctica: Servidor DHCP

## Tarea 1
> Escenario:
>
>- Servidor:
    - NAT para dar salida a Internet *(la tratamos como si fuera una pública)*
    - Privada1 (veryisolated)
    - Privada2 (veryisolated)
- nodo_lan1: cliente conectado a privada1
- nodo_lan2: cliente conectado a privada2
- nodowin10: cliente conectado a privada1

```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :servidor do |servidor|
    servidor.vm.box = "debian/bullseye64"
    servidor.vm.hostname = "servidor"
    servidor.vm.network :private_network,
      :libvirt__network_name => "practica-dhcp-nat",
      :ip => "192.168.0.2"
    servidor.vm.network :private_network,
      :libvirt__network_name => "practica-dhcp-privada1",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.100.1",
      :libvirt__forward_mode => "veryisolated"
    servidor.vm.network :private_network,
      :libvirt__network_name => "practica-dhcp-privada2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.200.1",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :nodolan1 do |nodolan1|
    nodolan1.vm.box = "debian/bullseye64"
    nodolan1.vm.hostname = "nodolan1"
    nodolan1.vm.network :private_network,
      :libvirt__network_name => "practica-dhcp-privada1",
      :libvirt__dhcp_enabled => false,
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :nodolan2 do |nodolan2|
    nodolan2.vm.box = "debian/bullseye64"
    nodolan2.vm.hostname = "nodolan2"
    nodolan2.vm.network :private_network,
      :libvirt__network_name => "practica-dhcp-privada2",
      :libvirt__dhcp_enabled => false,
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :nodowin10 do |nodowin10|
    nodowin10.vm.box = "jborean93/WindowsServer2016"
    nodowin10.vm.guest = :windows
    nodowin10.vm.hostname = "nodowin10"
    nodowin10.vm.communicator = "winrm"
    nodowin10.vm.network :private_network,
      :libvirt__network_name => "practica-dhcp-privada1",
      :libvirt__dhcp_enabled => false,
      :libvirt__forward_mode => "veryisolated"
  end

end
```

Para conectarnos a `nodowin10` hacemos:
```
vagrant ssh nodowin10
```


## Tarea 2
> Instalación servidor DHCP

```
sudo apt update && sudo apt install isc-dhcp-server
```

> Configuración DHCP

En `/etc/default/isc-dhcp-server`:

- Descomentar la línea: `DHCPDv4_CONF=/etc/dhcp/dhcpd.conf`
- Especificar interfaces: `INTERFACESv4="eth2 eth3"`  

En `/etc/dhcp/dhcpd.conf`:
```
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.100 192.168.100.150;
  option subnet-mask 255.255.255.0;
  option routers 192.168.100.1;
  option domain-name-servers 1.1.1.1, 1.0.0.1;
  default-lease-time 43200;
  max-lease-time 43200;
}

subnet 192.168.200.0 netmask 255.255.255.0 {
  range 192.168.200.100 192.168.200.150;
  option subnet-mask 255.255.255.0;
  option routers 192.168.200.1;
  option domain-name-servers 1.1.1.1, 1.0.0.1;
  default-lease-time 3600;
  max-lease-time 3600;
}
```

Reiniciar el servicio:
```
sudo systemctl restart isc-dhcp-server
```

> Listar concesiones y comprobación

Nodolan1 concesión:
```
lease 192.168.100.101 {
  starts 3 2021/10/20 06:40:30;
  ends 3 2021/10/20 18:40:30;
  cltt 3 2021/10/20 06:40:30;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 52:54:00:57:5b:cf;
  client-hostname "nodolan1";
}
```

Nodolan1 comprobación:
```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:57:5b:cf brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet 192.168.100.101/24 brd 192.168.100.255 scope global dynamic eth1
       valid_lft 41316sec preferred_lft 41316sec
    inet6 fe80::5054:ff:fe57:5bcf/64 scope link
       valid_lft forever preferred_lft forever
```

Nodolan2 concesión:
```
lease 192.168.200.101 {
  starts 3 2021/10/20 07:15:53;
  ends 3 2021/10/20 08:15:53;
  cltt 3 2021/10/20 07:15:53;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 52:54:00:eb:66:51;
  client-hostname "nodolan2";
}
```

Nodolan2 comprobación:
```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:eb:66:51 brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet 192.168.200.101/24 brd 192.168.200.255 scope global dynamic eth1
       valid_lft 3599sec preferred_lft 3599sec
    inet6 fe80::5054:ff:feeb:6651/64 scope link
       valid_lft forever preferred_lft forever
```

Nodowin10 concesión:
```
lease 192.168.100.102 {
  starts 3 2021/10/20 07:37:44;
  ends 3 2021/10/20 19:37:44;
  cltt 3 2021/10/20 07:37:44;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 52:54:00:57:de:32;
  uid "\001RT\000W\3362";
  set vendor-class-identifier = "MSFT 5.0";
  client-hostname "nodowin10";
}
```

Nodowin10 comprobación:
```
Ethernet adapter Ethernet 2:                                                                            

   Connection-specific DNS Suffix  . :                                                                  
   Description . . . . . . . . . . . : Red Hat VirtIO Ethernet Adapter #2                               
   Physical Address. . . . . . . . . : 52-54-00-57-DE-32                                                
   DHCP Enabled. . . . . . . . . . . : Yes                                                              
   Autoconfiguration Enabled . . . . : Yes                                                              
   Link-local IPv6 Address . . . . . : fe80::c1c4:20a1:511b:ecc8%2(Preferred)                           
   IPv4 Address. . . . . . . . . . . : 192.168.100.102(Preferred)                                       
   Subnet Mask . . . . . . . . . . . : 255.255.255.0                                                    
   Lease Obtained. . . . . . . . . . : Wednesday, October 20, 2021 7:37:44 AM                           
   Lease Expires . . . . . . . . . . : Wednesday, October 20, 2021 7:37:43 PM                           
   Default Gateway . . . . . . . . . : 192.168.100.1                                                    
   DHCP Server . . . . . . . . . . . : 192.168.100.1                                                    
   DHCPv6 IAID . . . . . . . . . . . : 72504320                                                         
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-29-01-8E-B1-52-54-00-CE-85-A1                        
   DNS Servers . . . . . . . . . . . : 1.1.1.1                                                          
                                       1.0.0.1                                                          
   NetBIOS over Tcpip. . . . . . . . : Enabled   
```

> Cambios en clientes para configuración DHCP

En `nodolan1`:

- Cambiar `/etc/network/interfaces`:

```
allow-hotplug eth0
iface eth0 inet dhcp
  post-up ip route del default dev $IFACE || true

auto eth1
iface eth1 inet dhcp
```

- Pedir configuración:  

```
sudo dhclient eth1
```

En `nodolan2`:

- Cambiar `/etc/network/interfaces`:

```
allow-hotplug eth0
iface eth0 inet dhcp
  post-up ip route del default dev $IFACE || true

auto eth1
iface eth1 inet dhcp
```
- Pedir configuración:

```
sudo dhclient eth1
```

En `nodowin10`:
```
ipconfig /renew "Ethernet 2"
```

## Tarea 3
> Servidor DHCP como router-NAT para que los clientes tengan internet

Hago router-nat y persistente:
```
sudo modprobe iptable_nat
echo "iptable_nat" >> /etc/modules
```
```
echo "1" > /proc/sys/net/ipv4/ip_forward
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
```
```
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
sudo apt install iptables-persistent
```

Cambio ruta por defecto para que el tráfico salga por la interfaz NAT que habíamos definido:
```
sudo ip route del default
sudo ip route add default via 192.168.0.1
```

Hago este cambio persistente modificando `/etc/network/interfaces`:
```
allow-hotplug eth0
iface eth0 inet dhcp
   post-up ip route del default dev $IFACE

auto eth1
iface eth1 inet static
      address 192.168.0.2
      netmask 255.255.255.0
      gateway 192.168.0.1
```
*(muestro __sólo__ los fragmentos modificados)*


> Mostrar rutas por defecto

En `servidor`
```
vagrant@servidor:~$ ip r
default via 192.168.0.1 dev eth1 onlink
192.168.0.0/24 dev eth1 proto kernel scope link src 192.168.0.2
192.168.100.0/24 dev eth2 proto kernel scope link src 192.168.100.1
192.168.121.0/24 dev eth0 proto kernel scope link src 192.168.121.164
192.168.200.0/24 dev eth3 proto kernel scope link src 192.168.200.1
```

En `nodolan1`
```
vagrant@nodolan1:~$ ip r
default via 192.168.100.1 dev eth1
192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.100
192.168.121.0/24 dev eth0 proto kernel scope link src 192.168.121.193
```

En `nodolan2`
```
vagrant@nodolan2:~$ ip r
default via 192.168.200.1 dev eth1
192.168.121.0/24 dev eth0 proto kernel scope link src 192.168.121.97
192.168.200.0/24 dev eth1 proto kernel scope link src 192.168.200.100
```

En `nodowin10`
```
Ethernet adapter Ethernet 2:                                                                            

   Connection-specific DNS Suffix  . :                                                                  
   Link-local IPv6 Address . . . . . : fe80::c1c4:20a1:511b:ecc8%2                                      
   IPv4 Address. . . . . . . . . . . : 192.168.100.102                                                  
   Subnet Mask . . . . . . . . . . . : 255.255.255.0                                                    
   Default Gateway . . . . . . . . . : 192.168.100.1    
```

> Pruebas de acceso a Internet en clientes

En `nodolan1`:

- ping

```
vagrant@nodolan1:~$ ping www.example.org
PING www.example.org (93.184.216.34) 56(84) bytes of data.
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=1 ttl=52 time=142 ms
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=2 ttl=52 time=160 ms
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=3 ttl=52 time=185 ms
^C
--- www.example.org ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 142.480/162.336/184.700/17.327 ms
```

- dig

```
vagrant@nodolan1:~$ dig www.example.org

; <<>> DiG 9.16.15-Debian <<>> www.example.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21798
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.example.org.		IN	A

;; ANSWER SECTION:
www.example.org.	82705	IN	A	93.184.216.34

;; Query time: 48 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Wed Oct 20 10:16:57 UTC 2021
;; MSG SIZE  rcvd: 60
```

En `nodolan2`:

- ping

```
vagrant@nodolan2:~$ ping www.example.org
PING www.example.org (93.184.216.34) 56(84) bytes of data.
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=1 ttl=52 time=143 ms
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=2 ttl=52 time=332 ms
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=3 ttl=52 time=193 ms

64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=4 ttl=52 time=142 ms
^C
--- www.example.org ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 4000ms
rtt min/avg/max/mdev = 142.189/202.596/332.117/77.558 ms
```

- dig

```
vagrant@nodolan2:~$ dig www.example.org

; <<>> DiG 9.16.15-Debian <<>> www.example.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4706
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.example.org.		IN	A

;; ANSWER SECTION:
www.example.org.	82395	IN	A	93.184.216.34

;; Query time: 472 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Wed Oct 20 10:22:02 UTC 2021
;; MSG SIZE  rcvd: 60
```

En `nodowin10`:

- ping

```
vagrant@NODOWIN10 C:\Users\vagrant>ping www.example.org                                                 

Pinging www.example.org [93.184.216.34] with 32 bytes of data:                                          
Reply from 93.184.216.34: bytes=32 time=141ms TTL=52                                                    
Reply from 93.184.216.34: bytes=32 time=149ms TTL=52                                                    
Reply from 93.184.216.34: bytes=32 time=141ms TTL=52                                                    
Reply from 93.184.216.34: bytes=32 time=142ms TTL=52                                                    

Ping statistics for 93.184.216.34:                                                                      
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),                                                
Approximate round trip times in milli-seconds:                                                          
    Minimum = 141ms, Maximum = 149ms, Average = 143ms   
```

- nslookup  

Hay 2 interfaces con configuración DNS, por lo que modifico la prioridad de interfaces. De esta manera me aseguro de que use los DNS que reparte **nuestro** servidor DHCP en las resoluciones.

Averiguar el "*InterfaceIndex*" de "*Ethernet 2*":
```
Get-NetIPInterface
```
```
ifIndex InterfaceAlias                  AddressFamily NlMtu(Bytes) InterfaceMetric Dhcp     ConnectionS
                                                                                            tate        
------- --------------                  ------------- ------------ --------------- ----     -----------
4       isatap.{05B72441-FF4C-475D-B... IPv6                  1280              75 Disabled Disconne...
2       Ethernet 2                      IPv6                  1500              15 Enabled  Connected   
5       Teredo Tunneling Pseudo-Inte... IPv6                  1280              75 Enabled  Connected   
7       Ethernet                        IPv6                  1500              15 Enabled  Connected   
3       isatap.{D6D1085A-1F02-4C69-A... IPv6                  1280              75 Disabled Disconne...
1       Loopback Pseudo-Interface 1     IPv6            4294967295              75 Disabled Connected   
2       Ethernet 2                      IPv4                  1500              15 Enabled  Connected   
7       Ethernet                        IPv4                  1500              15 Enabled  Connected   
1       Loopback Pseudo-Interface 1     IPv4            4294967295              75 Disabled Connected   
```

Modificamos su prioridad:
```
Set-NetIPInterface -InterfaceIndex 2 -InterfaceMetric 10
```

Vemos que la prioridad ha cambiado:
```
PS C:\Users\vagrant> Get-NetIPInterface                                                                 

ifIndex InterfaceAlias                  AddressFamily NlMtu(Bytes) InterfaceMetric Dhcp     ConnectionS
                                                                                            tate        
------- --------------                  ------------- ------------ --------------- ----     -----------
4       isatap.{05B72441-FF4C-475D-B... IPv6                  1280              75 Disabled Disconne...
2       Ethernet 2                      IPv6                  1500              10 Enabled  Connected   
5       Teredo Tunneling Pseudo-Inte... IPv6                  1280              75 Enabled  Connected   
7       Ethernet                        IPv6                  1500              15 Enabled  Connected   
3       isatap.{D6D1085A-1F02-4C69-A... IPv6                  1280              75 Disabled Disconne...
1       Loopback Pseudo-Interface 1     IPv6            4294967295              75 Disabled Connected   
2       Ethernet 2                      IPv4                  1500              10 Enabled  Connected   
7       Ethernet                        IPv4                  1500              15 Enabled  Connected   
1       Loopback Pseudo-Interface 1     IPv4            4294967295              75 Disabled Connected   
```

Hacemos la consulta DNS:
```
PS C:\Users\vagrant> nslookup www.example.org                                                           
Server:  one.one.one.one                                                                                
Address:  1.1.1.1                                                                                       

Non-authoritative answer:                                                                               
Name:    www.example.org                                                                                
Addresses:  2606:2800:220:1:248:1893:25c8:1946                                                          
          93.184.216.34   
```


## Tarea 4
> Con los clientes configurados apagamos el servidor DHCP. ¿Qué ocurrirá tanto en Linux como en Windows?

De ahora en adelante, `default-lease-time` y `max-lease-time` serán de 10 segundos para realizar esta tarea y la siguiente.

Así queda:
```
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.100 192.168.100.150;
  option subnet-mask 255.255.255.0;
  option routers 192.168.100.1;
  option domain-name-servers 1.1.1.1, 1.0.0.1;
  default-lease-time 10;
  max-lease-time 10;
}

subnet 192.168.200.0 netmask 255.255.255.0 {
  range 192.168.200.100 192.168.200.150;
  option subnet-mask 255.255.255.0;
  option routers 192.168.200.1;
  option domain-name-servers 1.1.1.1, 1.0.0.1;
  default-lease-time 10;
  max-lease-time 10;
}
```

**Tenemos que reiniciar el servicio** para que se apliquen estos cambios:
```
sudo systemctl restart isc-dhcp-server
```

Después de esto los clientes no actualizarán sus tiempos hasta que no pase el tiempo indicado, pero forzamos esto soltando el lease y pidiendo uno nuevo.

En los clientes linux hacemos:
```
sudo dhclient -r eth1
sudo dhclient eth1
```

En el cliente Windows hacemos:
```
ipconfig /release "Ethernet 2"
ipconfig /renew "Ethernet 2"
```

Ya teniendo los tiempos actualizados en todos los clientes, procedo a apagar el servidor.


Los clientes linux pierden la configuración por completo:
```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:57:5b:cf brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet6 fe80::5054:ff:fe57:5bcf/64 scope link
       valid_lft forever preferred_lft forever
```

El cliente Windows pierde la configuración y se autoconfigura con APIPA:
```
Ethernet adapter Ethernet 2:                                                                  

   Connection-specific DNS Suffix  . :                                                        
   Description . . . . . . . . . . . : Red Hat VirtIO Ethernet Adapter #2                     
   Physical Address. . . . . . . . . : 52-54-00-57-DE-32                                      
   DHCP Enabled. . . . . . . . . . . : Yes                                                    
   Autoconfiguration Enabled . . . . : Yes                                                    
   Link-local IPv6 Address . . . . . : fe80::c1c4:20a1:511b:ecc8%2(Preferred)                 
   Autoconfiguration IPv4 Address. . : 169.254.236.200(Preferred)                             
   Subnet Mask . . . . . . . . . . . : 255.255.0.0                                            
   Default Gateway . . . . . . . . . :                                                        
   DHCPv6 IAID . . . . . . . . . . . : 72504320                                               
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-29-01-8E-B1-52-54-00-CE-85-A1              
   DNS Servers . . . . . . . . . . . : fec0:0:0:ffff::1%1                                     
                                       fec0:0:0:ffff::2%1                                     
                                       fec0:0:0:ffff::3%1                                     
   NetBIOS over Tcpip. . . . . . . . : Enabled    
```


## Tarea 5
> Con los clientes configurados cambiamos la configuración del servidor DHCP (rango). ¿Qué ocurrirá tanto en Linux como en Windows?

Nuevos rangos:
```
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.200 192.168.100.210;
  option subnet-mask 255.255.255.0;
  option routers 192.168.100.1;
  option domain-name-servers 1.1.1.1, 1.0.0.1;
  default-lease-time 10;
  max-lease-time 10;
}

subnet 192.168.200.0 netmask 255.255.255.0 {
  range 192.168.200.200 192.168.200.210;
  option subnet-mask 255.255.255.0;
  option routers 192.168.200.1;
  option domain-name-servers 1.1.1.1, 1.0.0.1;
  default-lease-time 10;
  max-lease-time 10;
}
```

**Tenemos que reiniciar el servicio** para que se apliquen estos cambios:
```
sudo systemctl restart isc-dhcp-server
```

Los clientes linux actualizan su configuración sin problema:
```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:57:5b:cf brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet 192.168.100.200/24 brd 192.168.100.255 scope global dynamic eth1
       valid_lft 9sec preferred_lft 9sec
    inet6 fe80::5054:ff:fe57:5bcf/64 scope link
       valid_lft forever preferred_lft forever
```

El cliente Windows actualiza su configuración sin problema también:
```
Ethernet adapter Ethernet 2:                                                                  

   Connection-specific DNS Suffix  . :                                                        
   Link-local IPv6 Address . . . . . : fe80::c1c4:20a1:511b:ecc8%2                            
   IPv4 Address. . . . . . . . . . . : 192.168.100.201                                        
   Subnet Mask . . . . . . . . . . . : 255.255.255.0                                          
   Default Gateway . . . . . . . . . : 192.168.100.1   
```

## Tarea 6
> Realizar un playbook en ansible que configure el servidor como:
>
>- DHCP
- Router-NAT

<https://github.com/adriasir123/servidor-dhcp-ansible>
