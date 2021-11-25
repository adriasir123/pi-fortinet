# Práctica: Servidores Web, Base de Datos y DNS en escenario

## Entrega

### Parte 1
> Entregar configuración de cliente DNS en cada servidor

#### En zeus

Añado a `/etc/dhcp/dhclient.conf`:
```
#############
# My config #
#############

supersede domain-name "adrianj.gonzalonazareno.org";
supersede domain-search "adrianj.gonzalonazareno.org";
supersede domain-name-servers 10.0.1.102;
```

#### En apolo

Añado a `/etc/dhcp/dhclient.conf`:
```
#############
# My config #
#############

supersede domain-name "adrianj.gonzalonazareno.org";
supersede domain-search "adrianj.gonzalonazareno.org";
supersede domain-name-servers 127.0.0.1;
```

#### En ares

`/etc/netplan/01-netcfg.yaml`:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
      dhcp6: false
      optional: true
```
*(elimino nameservers)*

`/etc/systemd/resolved.conf`:
```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See resolved.conf(5) for details

[Resolve]
DNS=10.0.1.102
Domains=adrianj.gonzalonazareno.org

#FallbackDNS=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=yes
#DNSOverTLS=no
#Cache=yes
DNSStubListener=no
#ReadEtcHosts=yes
```
*(este fichero modifica `/etc/resolv.conf`)*

Reinicio `systemd-resolved`:
```
sudo systemctl restart systemd-resolved
```
*(esto actualiza `/etc/resolv.conf`)*

**¡Atención!**  
Por un error de systemd con `DNSStubListener=no` hay que actualizar manualmente el enlace simbólico, porque no lo hace automáticamente:
```
sudo rm /etc/resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

#### En hera

`/etc/NetworkManager/conf.d/90-dns-none.conf`:
```
[main]
dns=none
```
*(evito que NetworkManager modifique `/etc/resolv.conf`)*

Reinicio:
```
sudo systemctl restart NetworkManager
```

Escribo mi configuración en `/etc/resolv.conf`:
```
domain adrianj.gonzalonazareno.org
search adrianj.gonzalonazareno.org.
nameserver 10.0.1.102
```

### Parte 2
> Entregar definición de vistas y ficheros de zona

Añadir a `named.conf.options`:
```
allow-recursion { any; };
recursion yes;
allow-query { any; };
```

Comentar en `named.conf`:
```
// include "/etc/bind/named.conf.default-zones";
```

Comentar en `zones.rfc1918`:
```
// zone "16.172.in-addr.arpa"  { type master; file "/etc/bind/db.empty"; };
```

`named.conf.local`:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization


view externa {

    match-clients { 172.22.0.0/16; 192.168.202.2/32; 192.168.1.0/24; };

    zone "adrianj.gonzalonazareno.org" {
        type master;
        file "/etc/bind/db.externa.adrianj.gonzalonazareno.org";
    };

};



view dmz {

    match-clients { 172.16.0.0/16; };

    zone "adrianj.gonzalonazareno.org" {
        type master;
        file "/etc/bind/db.dmz.adrianj.gonzalonazareno.org";
    };

    zone "16.172.in-addr.arpa" {
        type master;
        file "/etc/bind/db.172.16";
    };

    zone "1.0.10.in-addr.arpa" {
        type master;
        file "/etc/bind/db.10.0.1";
    };

    include "/etc/bind/zones.rfc1918";
    include "/etc/bind/named.conf.default-zones";

};



view interna {

    match-clients { 10.0.1.0/24; 127.0.0.1/32; };

    zone "adrianj.gonzalonazareno.org" {
        type master;
        file "/etc/bind/db.interna.adrianj.gonzalonazareno.org";
    };

    zone "1.0.10.in-addr.arpa" {
        type master;
        file "/etc/bind/db.10.0.1";
    };

    zone "16.172.in-addr.arpa" {
        type master;
        file "/etc/bind/db.172.16";
    };


    include "/etc/bind/zones.rfc1918";
    include "/etc/bind/named.conf.default-zones";

};
```
*(en la vista externa he añadido la red de mi casa)*


`db.externa.adrianj.gonzalonazareno.org`:
```
$ORIGIN adrianj.gonzalonazareno.org.
$TTL 86400
@     IN     SOA    zeus.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      zeus.adrianj.gonzalonazareno.org.

; zeus           IN    A       172.22.0.213 ; clase
zeus           IN    A       192.168.1.106 ; casa-eth
www            IN    CNAME   zeus
```
*(la IP externa cambia según esté en clase o en casa)*

`db.dmz.adrianj.gonzalonazareno.org`:
```
$ORIGIN adrianj.gonzalonazareno.org.
$TTL 86400
@     IN     SOA    apolo.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      apolo.adrianj.gonzalonazareno.org.

zeus           IN    A       172.16.0.1
hera           IN    A       172.16.0.200
apolo          IN    A       10.0.1.102
ares           IN    A       10.0.1.101

www            IN    CNAME   hera
bd             IN    CNAME   ares
```

`db.interna.adrianj.gonzalonazareno.org`:
```
$ORIGIN adrianj.gonzalonazareno.org.
$TTL 86400
@     IN     SOA    apolo.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      apolo.adrianj.gonzalonazareno.org.

zeus           IN    A       10.0.1.1
apolo          IN    A       10.0.1.102
ares           IN    A       10.0.1.101
hera           IN    A       172.16.0.200

www            IN    CNAME   hera
bd             IN    CNAME   ares
```

`db.172.16`:
```
$ORIGIN 16.172.in-addr.arpa.
$TTL 86400
@     IN     SOA    apolo.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062502 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@                            IN     NS     apolo.adrianj.gonzalonazareno.org.

1.0                          IN     PTR    zeus.adrianj.gonzalonazareno.org.
200.0                        IN     PTR    hera.adrianj.gonzalonazareno.org.
```

`db.10.0.1`:
```
$ORIGIN 1.0.10.in-addr.arpa.
$TTL 86400
@     IN     SOA    apolo.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@        IN     NS     apolo.adrianj.gonzalonazareno.org.

1        IN     PTR    zeus.adrianj.gonzalonazareno.org.
102      IN     PTR    apolo.adrianj.gonzalonazareno.org.
101      IN     PTR    ares.adrianj.gonzalonazareno.org.
```

### Parte 3
> Consultas desde un cliente externo

*(en estas consultas usaré `@192.168.1.106` porque en mi casa no existe delegación del dominio)*

```
dig @192.168.1.106 NS adrianj.gonzalonazareno.org
```
```
; <<>> DiG 9.16.15-Debian <<>> @192.168.1.106 NS adrianj.gonzalonazareno.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58348
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e4468067cae8269e01000000619ff86a197a3852650fd14f (good)
;; QUESTION SECTION:
;adrianj.gonzalonazareno.org.	IN	NS

;; ANSWER SECTION:
adrianj.gonzalonazareno.org. 86400 IN	NS	zeus.adrianj.gonzalonazareno.org.

;; ADDITIONAL SECTION:
zeus.adrianj.gonzalonazareno.org. 86400	IN A	192.168.1.106

;; Query time: 0 msec
;; SERVER: 192.168.1.106#53(192.168.1.106)
;; WHEN: Thu Nov 25 21:56:10 CET 2021
;; MSG SIZE  rcvd: 119
```

```
dig @192.168.1.106 zeus.adrianj.gonzalonazareno.org
```
```
; <<>> DiG 9.16.15-Debian <<>> @192.168.1.106 zeus.adrianj.gonzalonazareno.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37293
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 84a54d72b820320701000000619ff98b8b9e767129e71df3 (good)
;; QUESTION SECTION:
;zeus.adrianj.gonzalonazareno.org. IN	A

;; ANSWER SECTION:
zeus.adrianj.gonzalonazareno.org. 86400	IN A	192.168.1.106

;; Query time: 0 msec
;; SERVER: 192.168.1.106#53(192.168.1.106)
;; WHEN: Thu Nov 25 22:00:59 CET 2021
;; MSG SIZE  rcvd: 105
```

```
dig @192.168.1.106 www.adrianj.gonzalonazareno.org
```
```
; <<>> DiG 9.16.15-Debian <<>> @192.168.1.106 www.adrianj.gonzalonazareno.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41074
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e1cddf11d68022f801000000619ff9cf5d2b25377cbac1b2 (good)
;; QUESTION SECTION:
;www.adrianj.gonzalonazareno.org. IN	A

;; ANSWER SECTION:
www.adrianj.gonzalonazareno.org. 86400 IN CNAME	zeus.adrianj.gonzalonazareno.org.
zeus.adrianj.gonzalonazareno.org. 86400	IN A	192.168.1.106

;; Query time: 0 msec
;; SERVER: 192.168.1.106#53(192.168.1.106)
;; WHEN: Thu Nov 25 22:02:07 CET 2021
;; MSG SIZE  rcvd: 123
```

```
dig @192.168.1.106 bd.adrianj.gonzalonazareno.org
```
```
; <<>> DiG 9.16.15-Debian <<>> @192.168.1.106 bd.adrianj.gonzalonazareno.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 40461
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a06ac8db7750971101000000619ffa4950b175b5a1d7d627 (good)
;; QUESTION SECTION:
;bd.adrianj.gonzalonazareno.org.	IN	A

;; AUTHORITY SECTION:
adrianj.gonzalonazareno.org. 86400 IN	SOA	zeus.adrianj.gonzalonazareno.org. admin.adrianj.gonzalonazareno.org. 2001062501 21600 3600 604800 86400

;; Query time: 4 msec
;; SERVER: 192.168.1.106#53(192.168.1.106)
;; WHEN: Thu Nov 25 22:04:09 CET 2021
;; MSG SIZE  rcvd: 134
```
*(no hay respuesta porque __bd no existe en la vista externa__)*

> Consultas desde un cliente interno

```
dig NS adrianj.gonzalonazareno.org
```
```
; <<>> DiG 9.16.1-Ubuntu <<>> NS adrianj.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17610
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 3f12cd4ba620cf1501000000619ffc618b1029d6fd6ee025 (good)
;; QUESTION SECTION:
;adrianj.gonzalonazareno.org.	IN	NS

;; ANSWER SECTION:
adrianj.gonzalonazareno.org. 86400 IN	NS	apolo.adrianj.gonzalonazareno.org.

;; ADDITIONAL SECTION:
apolo.adrianj.gonzalonazareno.org. 86400 IN A	10.0.1.102

;; Query time: 24 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Nov 25 21:13:05 UTC 2021
;; MSG SIZE  rcvd: 120
```

```
dig zeus.adrianj.gonzalonazareno.org
```
```
; <<>> DiG 9.16.1-Ubuntu <<>> zeus.adrianj.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15547
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 6268af10a08da00101000000619ffd1d5a37dcbdf9bcad63 (good)
;; QUESTION SECTION:
;zeus.adrianj.gonzalonazareno.org. IN	A

;; ANSWER SECTION:
zeus.adrianj.gonzalonazareno.org. 86400	IN A	10.0.1.1

;; Query time: 4 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Nov 25 21:16:13 UTC 2021
;; MSG SIZE  rcvd: 105
```

```
dig www.adrianj.gonzalonazareno.org
```
```
; <<>> DiG 9.16.1-Ubuntu <<>> www.adrianj.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35003
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ddc4f5521915c21901000000619ffd8ec03f6e53f09e84af (good)
;; QUESTION SECTION:
;www.adrianj.gonzalonazareno.org. IN	A

;; ANSWER SECTION:
www.adrianj.gonzalonazareno.org. 86400 IN CNAME	hera.adrianj.gonzalonazareno.org.
hera.adrianj.gonzalonazareno.org. 86400	IN A	172.16.0.200

;; Query time: 0 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Nov 25 21:18:06 UTC 2021
;; MSG SIZE  rcvd: 123
```

```
dig bd.adrianj.gonzalonazareno.org
```
```
; <<>> DiG 9.16.1-Ubuntu <<>> bd.adrianj.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3419
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 022009393ab949eb01000000619ffdec08dac774ea4bf941 (good)
;; QUESTION SECTION:
;bd.adrianj.gonzalonazareno.org.	IN	A

;; ANSWER SECTION:
bd.adrianj.gonzalonazareno.org.	86400 IN CNAME	ares.adrianj.gonzalonazareno.org.
ares.adrianj.gonzalonazareno.org. 86400	IN A	10.0.1.101

;; Query time: 4 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Nov 25 21:19:40 UTC 2021
;; MSG SIZE  rcvd: 122
```

> Consultas inversas en cada una de las redes *(interna+dmz+externa)*

En red interna:
```
dig -x 10.0.1.1
```
```
; <<>> DiG 9.16.1-Ubuntu <<>> -x 10.0.1.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4421
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 0d9da3496030523401000000619fff3ccb3de97da07c83e9 (good)
;; QUESTION SECTION:
;1.1.0.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
1.1.0.10.in-addr.arpa.	86400	IN	PTR	zeus.adrianj.gonzalonazareno.org.

;; Query time: 8 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Nov 25 21:25:16 UTC 2021
;; MSG SIZE  rcvd: 124
```

En red DMZ:
```
dig -x 172.16.0.1
```
```
; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> -x 172.16.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58791
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7c0c4c54f5e88ddd01000000619fffb8fd24258dd7580093 (good)
;; QUESTION SECTION:
;1.0.16.172.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
1.0.16.172.in-addr.arpa. 86400	IN	PTR	zeus.adrianj.gonzalonazareno.org.

;; Query time: 3 msec
;; SERVER: 10.0.1.102#53(10.0.1.102)
;; WHEN: Thu Nov 25 21:27:20 UTC 2021
;; MSG SIZE  rcvd: 126
```

En red externa:
```
dig @192.168.1.106 -x 10.0.1.101
```
```
; <<>> DiG 9.16.15-Debian <<>> @192.168.1.106 -x 10.0.1.101
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 27352
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c10440dae46509270100000061a00236623d14bb17cbdef9 (good)
;; QUESTION SECTION:
;101.1.0.10.in-addr.arpa.	IN	PTR

;; AUTHORITY SECTION:
10.IN-ADDR.ARPA.	86400	IN	SOA	10.IN-ADDR.ARPA. . 0 28800 7200 604800 86400

;; Query time: 0 msec
;; SERVER: 192.168.1.106#53(192.168.1.106)
;; WHEN: Thu Nov 25 22:37:58 CET 2021
;; MSG SIZE  rcvd: 130
```
*(no responde porque __en la vista externa no existen zonas de resolución inversa__)*

### Parte 4
> Captura de `www.adrianj.gonzalonazareno.org/info.php`

![](https://i.imgur.com/IUBf1QZ.png)

### Parte 5
> Entregar conexión a MariaDB desde hera

![](https://i.imgur.com/LSK21vk.png)

















## Servidor DNS (apolo)

### Parte 1
> DNAT en zeus redirigiendo puerto 53 a apolo

```
sudo iptables -t nat -A PREROUTING -i eth1 -p udp --dport 53 -j DNAT --to-destination 10.0.1.102:53
```
*(regla para clase)*

> DNAT en zeus redirigiendo puerto 80 a hera

```
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j DNAT --to-destination 172.16.0.200:80
```
*(regla para clase)*

> Guardar reglas

```
iptables-save > /etc/iptables/rules.v4
```

### Parte 2
> Configurar todos los servidores para resolver con apolo

#### En zeus

Añado a `/etc/dhcp/dhclient.conf`:
```
#############
# My config #
#############

supersede domain-name "adrianj.gonzalonazareno.org";
supersede domain-search "adrianj.gonzalonazareno.org";
supersede domain-name-servers 10.0.1.102;
```

#### En apolo

Añado a `/etc/dhcp/dhclient.conf`:
```
#############
# My config #
#############

supersede domain-name "adrianj.gonzalonazareno.org";
supersede domain-search "adrianj.gonzalonazareno.org";
supersede domain-name-servers 127.0.0.1;
```

#### En ares

`/etc/netplan/01-netcfg.yaml`:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
      dhcp6: false
      optional: true
```
*(elimino nameservers)*

`/etc/systemd/resolved.conf`:
```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See resolved.conf(5) for details

[Resolve]
DNS=10.0.1.102
Domains=adrianj.gonzalonazareno.org

#FallbackDNS=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=yes
#DNSOverTLS=no
#Cache=yes
DNSStubListener=no
#ReadEtcHosts=yes
```
*(este fichero modifica `/etc/resolv.conf`)*

Reinicio `systemd-resolved`:
```
sudo systemctl restart systemd-resolved
```
*(esto actualiza `/etc/resolv.conf`)*

**¡Atención!**  
Por un error de systemd con `DNSStubListener=no` hay que actualizar manualmente el enlace simbólico, porque no lo hace automáticamente:
```
sudo rm /etc/resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

#### En hera

`/etc/NetworkManager/conf.d/90-dns-none.conf`:
```
[main]
dns=none
```
*(evito que NetworkManager modifique `/etc/resolv.conf`)*

Reinicio:
```
sudo systemctl restart NetworkManager
```

Escribo mi configuración en `/etc/resolv.conf`:
```
domain adrianj.gonzalonazareno.org
search adrianj.gonzalonazareno.org.
nameserver 10.0.1.102
```

### Parte 3
> Configurar bind9

Añadir a `named.conf.options`:
```
allow-recursion { any; };
recursion yes;
allow-query { any; };
```

Comentar en `named.conf`:
```
// include "/etc/bind/named.conf.default-zones";
```

Comentar en `zones.rfc1918`:
```
// zone "16.172.in-addr.arpa"  { type master; file "/etc/bind/db.empty"; };
```

`named.conf.local`:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization


view externa {

    match-clients { 172.22.0.0/16; 192.168.202.2/32; 192.168.1.0/24; };

    zone "adrianj.gonzalonazareno.org" {
        type master;
        file "/etc/bind/db.externa.adrianj.gonzalonazareno.org";
    };

};



view dmz {

    match-clients { 172.16.0.0/16; };

    zone "adrianj.gonzalonazareno.org" {
        type master;
        file "/etc/bind/db.dmz.adrianj.gonzalonazareno.org";
    };

    zone "16.172.in-addr.arpa" {
        type master;
        file "/etc/bind/db.172.16";
    };

    zone "1.0.10.in-addr.arpa" {
        type master;
        file "/etc/bind/db.10.0.1";
    };

    include "/etc/bind/zones.rfc1918";
    include "/etc/bind/named.conf.default-zones";

};



view interna {

    match-clients { 10.0.1.0/24; 127.0.0.1/32; };

    zone "adrianj.gonzalonazareno.org" {
        type master;
        file "/etc/bind/db.interna.adrianj.gonzalonazareno.org";
    };

    zone "1.0.10.in-addr.arpa" {
        type master;
        file "/etc/bind/db.10.0.1";
    };

    zone "16.172.in-addr.arpa" {
        type master;
        file "/etc/bind/db.172.16";
    };


    include "/etc/bind/zones.rfc1918";
    include "/etc/bind/named.conf.default-zones";

};
```
*(en la vista externa he añadido la red de mi casa)*


`db.externa.adrianj.gonzalonazareno.org`:
```
$ORIGIN adrianj.gonzalonazareno.org.
$TTL 86400
@     IN     SOA    zeus.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      zeus.adrianj.gonzalonazareno.org.

; zeus           IN    A       172.22.0.213 ; clase
zeus           IN    A       192.168.1.106 ; casa-eth
www            IN    CNAME   zeus
```
*(la IP externa cambia según esté en clase o en casa)*

`db.dmz.adrianj.gonzalonazareno.org`:
```
$ORIGIN adrianj.gonzalonazareno.org.
$TTL 86400
@     IN     SOA    apolo.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      apolo.adrianj.gonzalonazareno.org.

zeus           IN    A       172.16.0.1
hera           IN    A       172.16.0.200
apolo          IN    A       10.0.1.102
ares           IN    A       10.0.1.101

www            IN    CNAME   hera
bd             IN    CNAME   ares
```

`db.interna.adrianj.gonzalonazareno.org`:
```
$ORIGIN adrianj.gonzalonazareno.org.
$TTL 86400
@     IN     SOA    apolo.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      apolo.adrianj.gonzalonazareno.org.

zeus           IN    A       10.0.1.1
apolo          IN    A       10.0.1.102
ares           IN    A       10.0.1.101
hera           IN    A       172.16.0.200

www            IN    CNAME   hera
bd             IN    CNAME   ares
```

`db.172.16`:
```
$ORIGIN 16.172.in-addr.arpa.
$TTL 86400
@     IN     SOA    apolo.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062502 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@                            IN     NS     apolo.adrianj.gonzalonazareno.org.

1.0                          IN     PTR    zeus.adrianj.gonzalonazareno.org.
200.0                        IN     PTR    hera.adrianj.gonzalonazareno.org.
```

`db.10.0.1`:
```
$ORIGIN 1.0.10.in-addr.arpa.
$TTL 86400
@     IN     SOA    apolo.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@        IN     NS     apolo.adrianj.gonzalonazareno.org.

1        IN     PTR    zeus.adrianj.gonzalonazareno.org.
102      IN     PTR    apolo.adrianj.gonzalonazareno.org.
101      IN     PTR    ares.adrianj.gonzalonazareno.org.
```






## Servidor Web (hera)

### Parte 1
> Instalar Apache

```
sudo dnf install httpd
systemctl enable --now  httpd
```

### Parte 2
> Instalar php + PHP-FPM

```
sudo dnf install php
```
*(instala PHP-FPM por dependencia)*

En `/var/www/www` descargo:
```
sudo wget https://gist.githubusercontent.com/SyntaxC4/5648247/raw/94277156638f9c309f2e36e19bff378ba7364907/info.php
```

### Parte 3
> Crear VirtualHost

`/etc/httpd/conf.d/www.conf`:
```
<VirtualHost *:80>

    ServerName www.adrianj.gonzalonazareno.org
    DocumentRoot /var/www/www

</VirtualHost>
```

### Parte 4
> Reiniciar Apache

```
sudo systemctl restart httpd
```

### Parte 5
> Instalar cliente MariaDB *(para entrega parte 5)*

```
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
sudo dnf install MariaDB-client
```





## Servidor de BD (ares)

### Parte 1
> Instalar MariaDB

```
sudo apt install mariadb-server
```

### Parte 2
> Habilitar conexiones remotas

`/etc/mysql/mariadb.conf.d/50-server.cnf`:
```
bind-address            = 0.0.0.0  
```

### Parte 3
> Habilitar root con acceso remoto

```
use mysql;
update user set plugin='' where User='root';
grant all privileges on *.* to 'root'@'%' identified by '1234';
flush privileges;
```

### Parte 4
> Reiniciar

```
sudo systemctl restart mariadb
```
