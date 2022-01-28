# Ejercicio 1 nftables: cortafuegos personal



## Preliminares

### Escenario

```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :ej1nftables1 do |ej1nftables1|
    ej1nftables1.vm.box = "debian/bullseye64"
    ej1nftables1.vm.hostname = "ej1nftables1"
    ej1nftables1.vm.network :private_network,
      :libvirt__network_name => "ej1-nftables-nodo",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :ej1nftables2 do |ej1nftables2|
    ej1nftables2.vm.box = "debian/bullseye64"
    ej1nftables2.vm.hostname = "ej1nftables2"
    ej1nftables2.vm.network :private_network,
      :libvirt__network_name => "ej1-nftables-nodo",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

Instalo un servidor web para dejar abierto el puerto 80:
```
sudo apt update
sudo apt install apache2
```



### Preparación nftables

Creo la tabla filter:
```
sudo nft add table inet filter
```

Creo las cadenas input/output en filter:
```
sudo nft add chain inet filter input { type filter hook input priority 0 \; counter \; policy accept \; }
sudo nft add chain inet filter output { type filter hook output priority 0 \; counter \; policy accept \; }
```



### Tráfico ssh entrante

```
sudo nft add rule inet filter input ip saddr 192.168.121.0/24 tcp dport 22 ct state new,established counter accept
sudo nft add rule inet filter output ip daddr 192.168.121.0/24 tcp sport 22 ct state established counter accept
```



### Cambiar política por defecto

```
sudo nft chain inet filter input { policy drop \; }
sudo nft chain inet filter output { policy drop \; }
```

Después de este paso me sigue funcionando la conexión SSH a la máquina, por lo que las reglas del paso anterior funcionan.

Todo tráfico diferente a este, no funciona por ahora.



### Tráfico loopback

```
sudo nft add rule inet filter output oifname "lo" counter accept
sudo nft add rule inet filter input iifname "lo" counter accept
```

Ya funciona el ping a localhost:
```
vagrant@ej1nftables1:~$ ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.025 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.073 ms
^C
--- 127.0.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1017ms
rtt min/avg/max/mdev = 0.025/0.049/0.073/0.024 ms
```



### Tráfico ICMP saliente

```
sudo nft add rule inet filter output oifname "eth0" icmp type echo-request counter accept
sudo nft add rule inet filter input iifname "eth0" icmp type echo-reply counter accept
```

Ya funciona el ping a Internet:
```
vagrant@ej1nftables1:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=368 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=135 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 135.491/251.525/367.559/116.034 ms
```



### Tráfico DNS saliente

```
sudo nft add rule inet filter output oifname "eth0" udp dport 53 ct state new,established counter accept
sudo nft add rule inet filter input iifname "eth0" udp sport 53 ct state established counter accept
```

Probamos que las consultas DNS funcionan:
```
vagrant@ej1nftables1:~$ dig @1.1.1.1 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @1.1.1.1 www.example.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 65305
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.example.org.	        IN	A

;; ANSWER SECTION:
www.example.org.	86400	IN	A	93.184.216.34

;; Query time: 132 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Jan 27 12:41:50 UTC 2022
;; MSG SIZE  rcvd: 60
```



### Tráfico http/https saliente

```
sudo nft add rule inet filter output oifname "eth0" ip protocol tcp tcp dport { 80,443 } ct state new,established counter accept
sudo nft add rule inet filter input iifname "eth0" ip protocol tcp tcp sport { 80,443 } ct state established counter accept
```

Compruebo que funciona http:
```
vagrant@ej1nftables1:~$ curl http://portquiz.net:80
Port test successful!
Your IP: 158.99.1.24
```

Compruebo que funciona https:
```
vagrant@ej1nftables1:~$ curl portquiz.net:443
Port test successful!
Your IP: 158.99.1.24
```



### Tráfico http entrante

```
sudo nft add rule inet filter input iifname "eth0" tcp dport 80 ct state new,established counter accept
sudo nft add rule inet filter output oifname "eth0" tcp sport 80 ct state established counter accept
```

Compruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej1-nftables-nodo$ telnet 192.168.121.65 80
Trying 192.168.121.65...
Connected to 192.168.121.65.
Escape character is '^]'.

```






## Ejercicios

### Ejercicio 1
> Permitir conexiones ssh al exterior

```
sudo nft add rule inet filter output tcp dport 22 ct state new,established counter accept
sudo nft add rule inet filter input tcp sport 22 ct state established counter accept
```

Pruebo que funciona conectando a mi host:
```
vagrant@ej1nftables1:~$ ssh atlas@192.168.121.1
The authenticity of host '192.168.121.1 (192.168.121.1)' can't be established.
ECDSA key fingerprint is SHA256:xbn++TBj5GyQdixTeO+cP7UGJzvMspD3AVK0JD1RbLw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.121.1' (ECDSA) to the list of known hosts.
atlas@192.168.121.1's password:
Linux olympus 5.10.0-8-amd64 #1 SMP Debian 5.10.46-4 (2021-08-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Wed Jan 26 10:32:49 2022 from 192.168.121.148
atlas@olympus:~$
```



### Ejercicio 2
> Denegar tráfico http entrante desde el host

```
sudo nft add rule inet filter input ip saddr 192.168.121.1 tcp dport 80 ct state new,established counter drop
```

Compruebo que bloquea el tráfico:
```
atlas@olympus:~/vagrant/ej1-nftables-nodo$ telnet 192.168.121.65 80
Trying 192.168.121.65...

```



### Ejercicio 3
> Permitir tráfico DNS saliente sólo a 192.168.202.2. Comprobar que no se puede hacer un dig @1.1.1.1

```
sudo nft add rule inet filter output ip daddr 192.168.202.2 udp dport 53 ct state new,established counter accept
sudo nft add rule inet filter input ip saddr 192.168.202.2 udp sport 53 ct state established counter accept
```

Pruebo que las consultas DNS a `192.168.202.2` funcionan:
```
vagrant@ej1nftables1:~$ dig @192.168.202.2 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @192.168.202.2 www.example.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18296
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 5

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: ce3cd730cac8cfacd646547461f3e6ed8a244c9fe1c24ed8 (good)
;; QUESTION SECTION:
;www.example.org.	        IN	A

;; ANSWER SECTION:
www.example.org.	86400	IN	A	93.184.216.34

;; AUTHORITY SECTION:
example.org.	        66421	IN	NS	a.iana-servers.net.
example.org.	        66421	IN	NS	b.iana-servers.net.

;; ADDITIONAL SECTION:
a.iana-servers.net.	152765	IN	A	199.43.135.53
b.iana-servers.net.	152765	IN	A	199.43.133.53
a.iana-servers.net.	152765	IN	AAAA	2001:500:8f::53
b.iana-servers.net.	152765	IN	AAAA	2001:500:8d::53

;; Query time: 492 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Fri Jan 28 12:51:57 UTC 2022
;; MSG SIZE  rcvd: 224
```

Pruebo que las consultas DNS a `1.1.1.1` no funcionan:
```
vagrant@ej1nftables1:~$ dig @1.1.1.1 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @1.1.1.1 www.example.org
; (1 server found)
;; global options: +cmd
;; connection timed out; no servers could be reached
```



### Ejercicio 4
> Bloquear tráfico http saliente a www.josedomingo.org (utilizar la ip). ¿Se puede acceder a fp.josedomingo.org?

```
sudo nft add rule inet filter output ip daddr 37.187.119.60 tcp dport 80 ct state new,established counter drop
```

Pruebo que no funciona el tráfico http saliente a `www.josedomingo.org`:
```
vagrant@ej1nftables1:~$ curl www.josedomingo.org

```
*(se queda esperando infinitamente porque no hay conexión)*

Pruebo a ver si funciona el tráfico http saliente a `fp.josedomingo.org`:
```
vagrant@ej1nftables1:~$ curl fp.josedomingo.org

```
*(Tampoco hay conexión. No funciona ya que `fp.josedomingo.org` se acaba traduciendo a la misma IP que hemos bloqueado antes)*

Pruebo que cualquier otro tráfico http saliente funciona:
```
vagrant@ej1nftables1:~$ telnet www.example.org 80
Trying 93.184.216.34...
Connected to www.example.org.
Escape character is '^]'.

```



### Ejercicio 5
> Permitir tráfico saliente a babuino-smtp.gonzalonazareno.org por el puerto 25

```
sudo nft add rule inet filter output ip daddr 192.168.203.3 tcp dport 25 ct state new,established counter accept
sudo nft add rule inet filter input ip saddr 192.168.203.3 tcp sport 25 ct state established counter accept
```

Pruebo que funciona con telnet:
```
vagrant@ej1nftables1:~$ telnet babuino-smtp.gonzalonazareno.org 25
Trying 192.168.203.3...
Connected to babuino-smtp.gonzalonazareno.org.
Escape character is '^]'.
220 babuino-smtp.gonzalonazareno.org ESMTP Postfix (Debian/GNU)

```



### Ejercicio 6

#### Parte 1
> Instalar mariadb-server y habilitar conexiones remotas

```
sudo apt install mariadb-server
```

`/etc/mysql/mariadb.conf.d/50-server.cnf`:
```
bind-address            = 0.0.0.0
```

Reinicio:
```
sudo systemctl restart mariadb
```

#### Parte 2
> Permitir tráfico entrante desde mi host

```
sudo nft add rule inet filter input ip saddr 192.168.121.1 tcp dport 3306 counter accept
sudo nft add rule inet filter output ip daddr 192.168.121.1 tcp sport 3306 counter accept
```

Pruebo que funciona:
```
atlas@olympus:~/vagrant/ej1-nftables-nodo$ nc -zvw10 192.168.121.65 3306
Connection to 192.168.121.65 3306 port [tcp/mysql] succeeded!
```

#### Parte 3
> Comprobar que desde otro cliente bloquea el tráfico

```
vagrant@ej1nftables2:~$ nc -zvw10 192.168.0.2 3306
192.168.0.2: inverse host lookup failed: Unknown host
(UNKNOWN) [192.168.0.2] 3306 (mysql) : Connection timed out
```
