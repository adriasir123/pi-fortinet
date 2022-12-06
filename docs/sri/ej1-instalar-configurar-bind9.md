# Ejercicio 1: Instalación y configuración del servidor bind9 en nuestra red local

## Entrega

### Parte 1
> Configurar `/etc/resolv.conf` en cliente apuntando a nuestro DNS

```
vagrant@cliente:~$ cat /etc/resolv.conf
nameserver 10.0.0.2
```

### Parte 2
> Entregar definición de zonas en `/etc/bind/named.conf.local`

```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
include "/etc/bind/zones.rfc1918";

zone "iesgn.org" {
    type master;
    file "/etc/bind/db.iesgn.org";
};

zone "0.0.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.0.0";
};
```

> Entregar zona directa `db.iesgn.org`

```
$ORIGIN iesgn.org.
$TTL 86400
@     IN     SOA    adrianj.iesgn.org.     admin.iesgn.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      adrianj.iesgn.org.
@              IN    MX  10  correo.iesgn.org.

adrianj        IN    A       10.0.0.2
correo         IN    A       10.0.0.200
ftp            IN    A       10.0.0.201

cliente1       IN    A       10.0.0.3
cliente2       IN    A       10.0.0.4
cliente3       IN    A       10.0.0.5

www            IN    CNAME   adrianj
departamentos  IN    CNAME   adrianj
```

> Entregar zona inversa `db.10.0.0`

```
$ORIGIN 0.0.10.in-addr.arpa.
$TTL 86400
@     IN     SOA    adrianj.iesgn.org.     admin.iesgn.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@     IN     NS     adrianj.iesgn.org.

2     IN     PTR    adrianj.iesgn.org.
200   IN     PTR    correo.iesgn.org.
201   IN     PTR    ftp.iesgn.org.

3     IN     PTR    cliente1.iesgn.org.
4     IN     PTR    cliente2.iesgn.org.
5     IN     PTR    cliente3.iesgn.org.
```

### Parte 3
> Realizar consultas DNS desde cliente

```
dig adrianj.iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> adrianj.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15723
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: cb6d28d55ed4ee5f0100000061943df81878244ad357cd14 (good)
;; QUESTION SECTION:
;adrianj.iesgn.org.		IN	A

;; ANSWER SECTION:
adrianj.iesgn.org.	86400	IN	A	10.0.0.2

;; Query time: 4 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Tue Nov 16 23:25:44 UTC 2021
;; MSG SIZE  rcvd: 90
```

```
dig www.iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> www.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12016
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 3d295ae1abc12a8e0100000061943e403a6b9d7c17e6de01 (good)
;; QUESTION SECTION:
;www.iesgn.org.			IN	A

;; ANSWER SECTION:
www.iesgn.org.		86400	IN	CNAME	adrianj.iesgn.org.
adrianj.iesgn.org.	86400	IN	A	10.0.0.2

;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Tue Nov 16 23:26:56 UTC 2021
;; MSG SIZE  rcvd: 108
```

```
dig ftp.iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> ftp.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58577
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: cffc03df6e9131a00100000061943e7b8644c93e1bbe2730 (good)
;; QUESTION SECTION:
;ftp.iesgn.org.			IN	A

;; ANSWER SECTION:
ftp.iesgn.org.		86400	IN	A	10.0.0.201

;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Tue Nov 16 23:27:55 UTC 2021
;; MSG SIZE  rcvd: 86
```

```
dig NS iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> NS iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19235
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: f8e47d8761996cfe0100000061943eb5c98b7df98b471892 (good)
;; QUESTION SECTION:
;iesgn.org.			IN	NS

;; ANSWER SECTION:
iesgn.org.		86400	IN	NS	adrianj.iesgn.org.

;; ADDITIONAL SECTION:
adrianj.iesgn.org.	86400	IN	A	10.0.0.2

;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Tue Nov 16 23:28:53 UTC 2021
;; MSG SIZE  rcvd: 104
```

```
dig MX iesgn.org
```
```
; <<>> DiG 9.16.22-Debian <<>> MX iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22032
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 01b2ad2987fc1e2d0100000061943efe0b5bff60f70d13d1 (good)
;; QUESTION SECTION:
;iesgn.org.			IN	MX

;; ANSWER SECTION:
iesgn.org.		86400	IN	MX	10 correo.iesgn.org.

;; ADDITIONAL SECTION:
correo.iesgn.org.	86400	IN	A	10.0.0.200

;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Tue Nov 16 23:30:06 UTC 2021
;; MSG SIZE  rcvd: 105
```

```
dig www.josedomingo.org
```
```
; <<>> DiG 9.16.22-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9813
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ef95496b3df04c9a0100000061943f350ccd2fde7ab6eda8 (good)
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	900	IN	CNAME	endor.josedomingo.org.
endor.josedomingo.org.	900	IN	A	37.187.119.60

;; Query time: 1136 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Tue Nov 16 23:31:01 UTC 2021
;; MSG SIZE  rcvd: 112
```

```
dig -x 10.0.0.2
```
```
; <<>> DiG 9.16.22-Debian <<>> -x 10.0.0.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51481
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ec34bd720d4b97710100000061943f724de5cec28b58d9ab (good)
;; QUESTION SECTION:
;2.0.0.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
2.0.0.10.in-addr.arpa.	86400	IN	PTR	adrianj.iesgn.org.

;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Tue Nov 16 23:32:02 UTC 2021
;; MSG SIZE  rcvd: 109
```





## Escenario

### Ejercicio 1
> Crear el escenario

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

end
```

### Ejercicio 2
> En `server`:
>
>- Instalar Apache
- 2 VirtualHosts
    - `www.iesgn.org`
    - `departamentos.iesgn.org`

```
sudo apt update
sudo apt install apache2
```

En `www.iesgn.org.conf`:
```
<VirtualHost *:80>

	ServerName www.iesgn.org

	DocumentRoot /var/www/www

</VirtualHost>
```

En `departamentos.iesgn.org.conf`:
```
<VirtualHost *:80>

	ServerName departamentos.iesgn.org

	DocumentRoot /var/www/departamentos

</VirtualHost>
```

Los habilito y reinicio:
```
sudo a2ensite www.iesgn.org.conf departamentos.iesgn.org.conf
sudo systemctl restart apache2
```

### Ejercicio 3
> En `server`, instalar `bind9`

```
sudo apt install bind9
```

### Ejercicio 4
> Cambiar el hostname de `server` a: `adrianj.iesgn.org`

Modifico `/etc/hostname`:
```
vagrant@server:~$ cat /etc/hostname
adrianj
```

Hago que el cambio de hostname tome efecto ahora:
```
sudo hostname adrianj
```

Verifico:
```
vagrant@server:~$ hostname
adrianj
```

Actualizo el prompt:
```
vagrant@server:~$ bash
vagrant@adrianj:~$
```

Modifico `/etc/hosts`:
```
127.0.0.1	localhost
127.0.0.2	bullseye

::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

127.0.1.1 adrianj.iesgn.org adrianj
```

Compruebo nombre largo:
```
vagrant@adrianj:~$ hostname -f
adrianj.iesgn.org
```

Compruebo nombre corto:
```
vagrant@adrianj:~$ ping adrianj -c 1
PING adrianj.iesgn.org (127.0.1.1) 56(84) bytes of data.
64 bytes from adrianj.iesgn.org (127.0.1.1): icmp_seq=1 ttl=64 time=0.056 ms

--- adrianj.iesgn.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.056/0.056/0.056/0.000 ms
```





## Servidor bind9

### Ejercicio 1
> Definición de zonas en `/etc/bind/named.conf.local`

```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
include "/etc/bind/zones.rfc1918";

zone "iesgn.org" {
    type master;
    file "/etc/bind/db.iesgn.org";
};

zone "0.0.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.0.0";
};
```

### Ejercicio 2
> Zona directa `db.iesgn.org`

```
$ORIGIN iesgn.org.
$TTL 86400
@     IN     SOA    adrianj.iesgn.org.     admin.iesgn.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      adrianj.iesgn.org.
@              IN    MX  10  correo.iesgn.org.

adrianj        IN    A       10.0.0.2
correo         IN    A       10.0.0.200
ftp            IN    A       10.0.0.201

cliente1       IN    A       10.0.0.3
cliente2       IN    A       10.0.0.4
cliente3       IN    A       10.0.0.5

www            IN    CNAME   adrianj
departamentos  IN    CNAME   adrianj
```

### Ejercicio 3
> Zona inversa `db.10.0.0`

```
$ORIGIN 0.0.10.in-addr.arpa.
$TTL 86400
@     IN     SOA    adrianj.iesgn.org.     admin.iesgn.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@     IN     NS     adrianj.iesgn.org.

2     IN     PTR    adrianj.iesgn.org.
200   IN     PTR    correo.iesgn.org.
201   IN     PTR    ftp.iesgn.org.

3     IN     PTR    cliente1.iesgn.org.
4     IN     PTR    cliente2.iesgn.org.
5     IN     PTR    cliente3.iesgn.org.
```

### Ejercicio 4
> Comprobar configuraciones y reiniciar el servidor

Verifico la zona directa:
```
sudo named-checkzone iesgn.org db.iesgn.org
```
```
zone iesgn.org/IN: loaded serial 2001062501
OK
```

Verifico la zona inversa:
```
sudo named-checkzone 0.0.10.in-addr.arpa db.10.0.0
```
```
zone 0.0.10.in-addr.arpa/IN: loaded serial 2001062501
OK
```

Reinicio:
```
sudo systemctl restart bind9
```
