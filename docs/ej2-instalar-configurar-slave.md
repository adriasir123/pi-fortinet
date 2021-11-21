# Ejercicio 2: Instalación y configuración de un servidor DNS esclavo

## Entrega

### Parte 1
> Mostrar la definición de zonas en maestro

`/etc/bind/named.conf.local`:
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
    allow-transfer { 10.0.0.10; };
    also-notify { 10.0.0.10; };
};

zone "0.0.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.0.0";
    allow-transfer { 10.0.0.10; };
    also-notify { 10.0.0.10; };
};
```

> Mostrar la zona directa en maestro

`/etc/bind/db.iesgn.org`:
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
@              IN    NS      jaramillor.iesgn.org.
@              IN    MX  10  correo.iesgn.org.

adrianj        IN    A       10.0.0.2
jaramillor     IN    A       10.0.0.10
correo         IN    A       10.0.0.200
ftp            IN    A       10.0.0.201

cliente1       IN    A       10.0.0.3
cliente2       IN    A       10.0.0.4
cliente3       IN    A       10.0.0.5

www            IN    CNAME   adrianj
departamentos  IN    CNAME   adrianj
```

> Mostrar la zona inversa en maestro

`/etc/bind/db.10.0.0`:
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
@     IN     NS     jaramillor.iesgn.org.

2     IN     PTR    adrianj.iesgn.org.
10    IN     PTR    jaramillor.iesgn.org.
200   IN     PTR    correo.iesgn.org.
201   IN     PTR    ftp.iesgn.org.

3     IN     PTR    cliente1.iesgn.org.
4     IN     PTR    cliente2.iesgn.org.
5     IN     PTR    cliente3.iesgn.org.
```

> Mostrar la definición de zonas en esclavo

`/etc/bind/named.conf.local`:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
include "/etc/bind/zones.rfc1918";

zone "iesgn.org" {
    type slave;
    file "/var/cache/bind/db.iesgn.org";
    masters { 10.0.0.2; };
};

zone "0.0.10.in-addr.arpa" {
    type slave;
    file "/var/cache/bind/db.10.0.0";
    masters { 10.0.0.2; };
};
```

### Parte 2
> Mostrar si los ficheros de zona del maestro tienen errores

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

### Parte 3
> Comprobar si `named.conf` tiene errores tanto en maestro como en esclavo

En maestro:
```
sudo named-checkconf named.conf
```

En esclavo:
```
sudo named-checkconf named.conf
```

No me devuelve output en ambos casos, significa que no hay errores.

### Parte 4
> Mostrar en `/var/log/syslog` la transferencia de zonas

Comando:
```
sudo cat /var/log/syslog | grep named | grep transfer
```

Transferencia de zona directa:
```
Nov 18 00:43:58 bullseye named[2538]: transfer of 'iesgn.org/IN' from 10.0.0.2#53: connected using 10.0.0.10#58761
Nov 18 00:43:58 bullseye named[2538]: zone iesgn.org/IN: transferred serial 2001062504
Nov 18 00:43:58 bullseye named[2538]: transfer of 'iesgn.org/IN' from 10.0.0.2#53: Transfer status: success
Nov 18 00:43:58 bullseye named[2538]: transfer of 'iesgn.org/IN' from 10.0.0.2#53: Transfer completed: 1 messages, 14 records, 364 bytes, 0.004 secs (91000 bytes/sec) (serial 2001062504)
```

Transferencia de zona inversa:
```
Nov 18 00:53:00 bullseye named[2538]: transfer of '0.0.10.in-addr.arpa/IN' from 10.0.0.2#53: connected using 10.0.0.10#58699
Nov 18 00:53:00 bullseye named[2538]: zone 0.0.10.in-addr.arpa/IN: transferred serial 2001062502
Nov 18 00:53:00 bullseye named[2538]: transfer of '0.0.10.in-addr.arpa/IN' from 10.0.0.2#53: Transfer status: success
Nov 18 00:53:00 bullseye named[2538]: transfer of '0.0.10.in-addr.arpa/IN' from 10.0.0.2#53: Transfer completed: 1 messages, 11 records, 326 bytes, 0.004 secs (81500 bytes/sec) (serial 2001062502)
```

### Parte 5
> Entregar la configuración DNS de cliente

`/etc/resolv.conf`:
```
nameserver 10.0.0.2
nameserver 10.0.0.10
```

### Parte 6
> Realizar una consulta dig a maestro y esclavo para comprobar que las respuestas son con autoridad

Al maestro:

![](https://i.imgur.com/oQi96hF.png)

Al esclavo:

![](https://i.imgur.com/RlMps2b.png)

Buscamos esa flag "aa", que quiere decir "Authoritative Answer".  
Nos devolverá esa flag siempre que hagamos consultas sobre zonas en las que los servidores tengan autoridad.

La [RFC 1035](https://www.ietf.org/rfc/rfc1035.txt) dice lo siguiente:

![](https://i.imgur.com/0iqpeYW.png)

Si por ejemplo hiciese una consulta en una zona sobre la que mi servidor *NO TIENE* autoridad, pasaría lo siguiente:

![](https://i.imgur.com/B6RLmkj.png)

No aparece la flag "aa".

### Parte 7
> Solicitar copia completa de una zona desde cliente ¿qué tiene que ocurrir?

```
dig @10.0.0.2 iesgn.org axfr
```
```
; <<>> DiG 9.16.22-Debian <<>> @10.0.0.2 iesgn.org axfr
; (1 server found)
;; global options: +cmd
; Transfer failed.
```
No permite la transferencia.

> Solicitar copia completa de una zona desde esclavo ¿qué tiene que ocurrir?

```
dig @10.0.0.2 iesgn.org axfr
```
```
; <<>> DiG 9.16.22-Debian <<>> @10.0.0.2 iesgn.org axfr
; (1 server found)
;; global options: +cmd
iesgn.org.		86400	IN	SOA	adrianj.iesgn.org. admin.iesgn.org. 2001062503 21600 3600 604800 86400
iesgn.org.		86400	IN	NS	adrianj.iesgn.org.
iesgn.org.		86400	IN	NS	jaramillor.iesgn.org.
iesgn.org.		86400	IN	MX	10 correo.iesgn.org.
adrianj.iesgn.org.	86400	IN	A	10.0.0.2
cliente1.iesgn.org.	86400	IN	A	10.0.0.3
cliente2.iesgn.org.	86400	IN	A	10.0.0.4
cliente3.iesgn.org.	86400	IN	A	10.0.0.5
correo.iesgn.org.	86400	IN	A	10.0.0.200
departamentos.iesgn.org. 86400	IN	CNAME	adrianj.iesgn.org.
ftp.iesgn.org.		86400	IN	A	10.0.0.201
jaramillor.iesgn.org.	86400	IN	A	10.0.0.10
www.iesgn.org.		86400	IN	CNAME	adrianj.iesgn.org.
iesgn.org.		86400	IN	SOA	adrianj.iesgn.org. admin.iesgn.org. 2001062503 21600 3600 604800 86400
;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Wed Nov 17 23:50:34 UTC 2021
;; XFR size: 14 records (messages 1, bytes 403)
```
Permite la transferencia.

### Parte 8
> Realizar consulta desde cliente y comprobar qué servidor responde

![](https://i.imgur.com/KkLDV39.png)

Según nuestro orden en `resolv.conf`, siempre nos responderá el maestro, salvo que éste falle.

### Parte 9
> Apagar maestro y hacer consulta desde cliente ¿quién responde?

![](https://i.imgur.com/VMsKXKA.png)

Responde el esclavo.






## Realización

### Ejercicio 1

Instalar un servidor DNS como esclavo del DNS configurado en el ejercicio 1.

#### Parte 1
> Añadir otra máquina al escenario del ej1, para instalar el esclavo ahí

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

end
```

#### Parte 2
> Cambiar el hostname de `slavedns` a: `jaramillor.iesgn.org`

Modifico `/etc/hostname`:
```
vagrant@slavedns:~$ cat /etc/hostname
jaramillor
```

Hago que el cambio de hostname tome efecto ahora:
```
sudo hostname jaramillor
```

Verifico:
```
vagrant@slavedns:~$ hostname
jaramillor
```

Actualizo el prompt:
```
vagrant@slavedns:~$ bash
vagrant@jaramillor:~$
```

Modifico `/etc/hosts`:
```
127.0.0.1	localhost
127.0.0.2	bullseye
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

127.0.1.1 jaramillor.iesgn.org jaramillor
```

Compruebo nombre largo:
```
vagrant@jaramillor:~$ hostname -f
jaramillor.iesgn.org
```

Compruebo nombre corto:
```
vagrant@jaramillor:~$ ping jaramillor -c 1
PING jaramillor.iesgn.org (127.0.1.1) 56(84) bytes of data.
64 bytes from jaramillor.iesgn.org (127.0.1.1): icmp_seq=1 ttl=64 time=0.089 ms

--- jaramillor.iesgn.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.089/0.089/0.089/0.000 ms
```

#### Parte 3
> Instalar `bind9`

```
sudo apt update
sudo apt install bind9
```

#### Parte 4
> Modificar la definición de zonas del maestro para habilitar transferencia al esclavo

`/etc/bind/named.conf.local`:
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
    allow-transfer { 10.0.0.10; };
    also-notify { 10.0.0.10; };
};

zone "0.0.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.0.0";
    allow-transfer { 10.0.0.10; };
    also-notify { 10.0.0.10; };
};
```

#### Parte 5
> Modificar la zona directa con el nuevo esclavo

`/etc/bind/db.iesgn.org`:
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
@              IN    NS      jaramillor.iesgn.org.
@              IN    MX  10  correo.iesgn.org.

adrianj        IN    A       10.0.0.2
jaramillor     IN    A       10.0.0.10
correo         IN    A       10.0.0.200
ftp            IN    A       10.0.0.201

cliente1       IN    A       10.0.0.3
cliente2       IN    A       10.0.0.4
cliente3       IN    A       10.0.0.5

www            IN    CNAME   adrianj
departamentos  IN    CNAME   adrianj
```

#### Parte 6
> Modificar la zona inversa con el nuevo esclavo

`/etc/bind/db.10.0.0`:
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
@     IN     NS     jaramillor.iesgn.org.

2     IN     PTR    adrianj.iesgn.org.
10    IN     PTR    jaramillor.iesgn.org.
200   IN     PTR    correo.iesgn.org.
201   IN     PTR    ftp.iesgn.org.

3     IN     PTR    cliente1.iesgn.org.
4     IN     PTR    cliente2.iesgn.org.
5     IN     PTR    cliente3.iesgn.org.
```

#### Parte 7
> Reiniciar DNS maestro

```
sudo systemctl restart bind9
```

#### Parte 8
> Modificar la definición de zonas del esclavo

`/etc/bind/named.conf.local`:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
include "/etc/bind/zones.rfc1918";

zone "iesgn.org" {
    type slave;
    file "/var/cache/bind/db.iesgn.org";
    masters { 10.0.0.2; };
};

zone "0.0.10.in-addr.arpa" {
    type slave;
    file "/var/cache/bind/db.10.0.0";
    masters { 10.0.0.2; };
};
```

#### Parte 9
> Reiniciar DNS esclavo

```
sudo systemctl restart bind9
```

### Ejercicio 2
> Comprobar si los ficheros de zona del maestro tienen errores

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

### Ejercicio 3
> Comprobar si `named.conf` tiene errores tanto en maestro como en esclavo

En maestro:
```
sudo named-checkconf named.conf
```

En esclavo:
```
sudo named-checkconf named.conf
```

### Ejercicio 4
> Reiniciar ambos servidores DNS

```
sudo systemctl restart bind9
```

> Comprobar en `/var/log/syslog` si hay algún error *(incrementar número de serie en SOA si se modifican zonas en el maestro)*

```
sudo cat /var/log/syslog | grep named
```

No hay errores.

### Ejercicio 5
> Configurar cliente para utilizar maestro y esclavo como servidores DNS

`/etc/resolv.conf`:
```
nameserver 10.0.0.2
nameserver 10.0.0.10
```
