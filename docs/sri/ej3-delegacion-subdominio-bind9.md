# Ejercicio 3: Delegación de subdominios con bind9

## Entrega

Realizar las siguientes consultas desde cliente.

### Parte 1

```
dig www.informatica.iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> www.informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45194
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 4b05b345b807917401000000619ad9b4073217deccc47a09 (good)
;; QUESTION SECTION:
;www.informatica.iesgn.org.	IN	A

;; ANSWER SECTION:
www.informatica.iesgn.org. 86400 IN	CNAME	sweb.informatica.iesgn.org.
sweb.informatica.iesgn.org. 86400 IN	A	10.0.0.52

;; Query time: 96 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Sun Nov 21 23:43:48 UTC 2021
;; MSG SIZE  rcvd: 117
```

```
dig ftp.informatica.iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> ftp.informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31505
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7897922eed5f0acb01000000619ada1198bb5211fb4cf940 (good)
;; QUESTION SECTION:
;ftp.informatica.iesgn.org.	IN	A

;; ANSWER SECTION:
ftp.informatica.iesgn.org. 86400 IN	CNAME	sweb.informatica.iesgn.org.
sweb.informatica.iesgn.org. 86307 IN	A	10.0.0.52

;; Query time: 4 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Sun Nov 21 23:45:21 UTC 2021
;; MSG SIZE  rcvd: 117
```

### Parte 2
```
dig NS informatica.iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> NS informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48818
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 0056b02be3bb838301000000619adae91b06c7f48ca54b49 (good)
;; QUESTION SECTION:
;informatica.iesgn.org.		IN	NS

;; ANSWER SECTION:
informatica.iesgn.org.	86400	IN	NS	adrianj-ns.informatica.iesgn.org.

;; Query time: 4 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Sun Nov 21 23:48:57 UTC 2021
;; MSG SIZE  rcvd: 103
```

> ¿Es el mismo que el servidor DNS con autoridad para la zona `iesgn.org`?

```
dig NS iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> NS iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36865
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7a0344b5113fcb0b01000000619adb55de76d31d7232e8de (good)
;; QUESTION SECTION:
;iesgn.org.			IN	NS

;; ANSWER SECTION:
iesgn.org.		86400	IN	NS	adrianj.iesgn.org.
iesgn.org.		86400	IN	NS	jaramillor.iesgn.org.

;; ADDITIONAL SECTION:
adrianj.iesgn.org.	86400	IN	A	10.0.0.2
jaramillor.iesgn.org.	86400	IN	A	10.0.0.10

;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Sun Nov 21 23:50:45 UTC 2021
;; MSG SIZE  rcvd: 145
```

No, no son los mismos servidores DNS.

### Parte 3
```
dig MX informatica.iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> MX informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22488
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 838d443349ddf15a01000000619adbeb9351de6157a5e8fa (good)
;; QUESTION SECTION:
;informatica.iesgn.org.		IN	MX

;; ANSWER SECTION:
informatica.iesgn.org.	86400	IN	MX	10 correo.informatica.iesgn.org.

;; Query time: 4 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Sun Nov 21 23:53:15 UTC 2021
;; MSG SIZE  rcvd: 101
```






## Realización

Este ejercicio parte del escenario usado en [Ejercicio 2: Slave DNS](https://conocimiento-en-bits.neocities.org/ej2-instalar-configurar-slave/#parte-1_1).

Se delegará el subdominio `informatica.iesgn.org` a `adrianj-ns.informatica.iesgn.org` *(se encuentra en otra máquina)*

### Ejercicio 1
> Añadir otra máquina al escenario del ej2, para instalar ahí el DNS al que delegaremos el subdominio

`Vagrantfile`:
```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :server do |server|
    server.vm.box = "debian/bullseye64"
    server.vm.hostname = "server"
    server.vm.network :private_network,
      :libvirt__network_name => "ej1bind9",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.2",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :cliente do |cliente|
    cliente.vm.box = "debian/bullseye64"
    cliente.vm.hostname = "cliente"
    cliente.vm.network :private_network,
      :libvirt__network_name => "ej1bind9",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.3",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :slavedns do |slavedns|
    slavedns.vm.box = "debian/bullseye64"
    slavedns.vm.hostname = "slavedns"
    slavedns.vm.network :private_network,
      :libvirt__network_name => "ej1bind9",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.10",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :subdominio do |subdominio|
    subdominio.vm.box = "debian/bullseye64"
    subdominio.vm.hostname = "subdominio"
    subdominio.vm.network :private_network,
      :libvirt__network_name => "ej1bind9",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.50",
      :libvirt__forward_mode => "veryisolated"
  end

end
```

### Ejercicio 2
> Cambiar el hostname de `subdominio` a: `adrianj-ns.informatica.iesgn.org`

Modifico `/etc/hostname`:
```
vagrant@subdominio:~$ cat /etc/hostname
adrianj-ns
```

Hago que el cambio de hostname tome efecto ahora:
```
sudo hostname adrianj-ns
```

Verifico:
```
vagrant@subdominio:~$ hostname
adrianj-ns
```

Actualizo el prompt:
```
vagrant@subdominio:~$ bash
vagrant@adrianj-ns:~$
```

Modifico `/etc/hosts`:
```
127.0.0.1	localhost
127.0.0.2	bullseye
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

127.0.1.1 adrianj-ns.informatica.iesgn.org adrianj-ns
```

Compruebo nombre largo:
```
vagrant@adrianj-ns:~$ hostname -f
adrianj-ns.informatica.iesgn.org
```

Compruebo nombre corto:
```
vagrant@adrianj-ns:~$ ping adrianj-ns -c 1
PING adrianj-ns.informatica.iesgn.org (127.0.1.1) 56(84) bytes of data.
64 bytes from adrianj-ns.informatica.iesgn.org (127.0.1.1): icmp_seq=1 ttl=64 time=0.015 ms

--- adrianj-ns.informatica.iesgn.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.015/0.015/0.015/0.000 ms
```

### Ejercicio 3
> Instalar `bind9`

```
sudo apt update
sudo apt install bind9
```

### Ejercicio 4
> Añadir la delegación de `informatica.iesgn.org` en `adrianj.iesgn.org`

`db.iesgn.org`:
```
$ORIGIN iesgn.org.
$TTL 86400
@     IN     SOA    adrianj.iesgn.org.     admin.iesgn.org. (
                    2001062505 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      adrianj.iesgn.org.
@              IN    NS      jaramillor.iesgn.org.
@              IN    MX  10  correo.iesgn.org.

adrianj        IN    A       10.0.0.2
jaramillor     IN    A       10.0.0.10
correo         IN    A       10.0.0.200
ftp            IN    A       10.0.0.201

cliente1       IN    A       10.0.0.3
cliente2       IN    A       10.0.0.4
cliente3       IN    A       10.0.0.6
cliente4       IN    A       10.0.0.7

www            IN    CNAME   adrianj
departamentos  IN    CNAME   adrianj


; Definición de subdominio

$ORIGIN informatica.iesgn.org.
@              IN    NS      adrianj-ns.informatica.iesgn.org.
adrianj-ns     IN    A       10.0.0.50 ; glue record
```

Reinicio:
```
sudo systemctl restart bind9
```

### Ejercicio 5
> Definir la zona `informatica.iesgn.org` en `adrianj-ns`

`named.conf.local`:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
// include "/etc/bind/zones.rfc1918";

zone "informatica.iesgn.org" {
    type master;
    file "/etc/bind/db.informatica.iesgn.org";
};
```

### Ejercicio 6
> Definir zona directa para `informatica.iesgn.org`

`db.informatica.iesgn.org`:
```
$ORIGIN informatica.iesgn.org.
$TTL 86400
@     IN     SOA    adrianj-ns.informatica.iesgn.org.     admin.informatica.iesgn.org. (
                    2001062505 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      adrianj-ns.informatica.iesgn.org.
@              IN    MX  10  correo.informatica.iesgn.org.

adrianj-ns     IN    A       10.0.0.50
correo         IN    A       10.0.0.51
sweb           IN    A       10.0.0.52

www            IN    CNAME   sweb
ftp            IN    CNAME   sweb
```

Reinicio:
```
sudo systemctl restart bind9
```

### Ejercicio 7
> Actualizar la zona inversa en `adrianj.iesgn.org` con las nuevas máquinas del subdominio

`db.10.0.0`:
```
$ORIGIN 0.0.10.in-addr.arpa.
$TTL 86400
@     IN     SOA    adrianj.iesgn.org.     admin.iesgn.org. (
                    2001062502 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@     IN     NS     adrianj.iesgn.org.
@     IN     NS     jaramillor.iesgn.org.

2     IN     PTR    adrianj.iesgn.org.
10    IN     PTR    jaramillor.iesgn.org.
200   IN     PTR    correo.iesgn.org.
201   IN     PTR    ftp.iesgn.org.

3     IN     PTR    cliente1.iesgn.org.
4     IN     PTR    cliente2.iesgn.org.
6     IN     PTR    cliente3.iesgn.org.

; Máquinas del subdominio

50    IN     PTR    adrianj-ns.informatica.iesgn.org.
51    IN     PTR    correo.informatica.iesgn.org.
52    IN     PTR    sweb.informatica.iesgn.org.
```

Reinicio:
```
sudo systemctl restart bind9
```
