# Ejercicio 2 nftables: cortafuegos perimetral



## Preliminares

### Escenario

```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :perimetral do |perimetral|
    perimetral.vm.box = "debian/bullseye64"
    perimetral.vm.hostname = "perimetral"
    perimetral.vm.network :public_network,
      :dev => "br0",
      :mode => "bridge",
      :type => "bridge"
    perimetral.vm.network :private_network,
      :libvirt__network_name => "ej2-nftables-perimetral",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.100.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :local do |local|
    local.vm.box = "debian/bullseye64"
    local.vm.hostname = "local"
    local.vm.network :private_network,
      :libvirt__network_name => "ej2-nftables-perimetral",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.100.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

Cambio la ruta por defecto del cortafuegos:
```
sudo ip route del default
sudo ip route add default via 172.22.0.1
```

Cambio la ruta por defecto de la máquina de la LAN:
```
sudo ip route del default
sudo ip route add default via 192.168.100.2
```

Instalo un servidor web y un servidor de correos en la máquina de la LAN para dejar abierto el puerto 80 y el 25:
```
sudo apt update
sudo apt install apache2 postfix
```



### Preparación nftables

Creo las tablas filter y nat:
```
sudo nft add table inet filter
sudo nft add table inet nat
```

Creo las cadenas de filter:
```
sudo nft add chain inet filter input { type filter hook input priority 0 \; counter \; policy accept \; }
sudo nft add chain inet filter output { type filter hook output priority 0 \; counter \; policy accept \; }
sudo nft add chain inet filter forward { type filter hook forward priority 0 \; counter \; policy accept \; }
```

Creo las cadenas de nat:
```
sudo nft add chain inet nat prerouting { type nat hook prerouting priority 0 \; }
sudo nft add chain inet nat postrouting { type nat hook postrouting priority 100 \; }
```



### Tráfico ssh entrante al cortafuegos

```
sudo nft add rule inet filter input ip saddr 192.168.121.0/24 tcp dport 22 ct state new,established counter accept
sudo nft add rule inet filter output ip daddr 192.168.121.0/24 tcp sport 22 ct state established counter accept
```



### Políticas por defecto

```
sudo nft chain inet filter input { policy drop \; }
sudo nft chain inet filter output { policy drop \; }
sudo nft chain inet filter forward { policy drop \; }
```

No puedo hacer ping a localhost:
```
vagrant@perimetral:~$ ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
^C
--- 127.0.0.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2049ms
```

No puedo hacer ping a Internet:
```
vagrant@perimetral:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
^C
--- 1.1.1.1 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3075ms
```

Si no se nos corta la conexión ssh a la máquina después de este paso, el par de reglas que aplicamos antes funcionan.



### Activar bit de forward

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```



### SNAT

```
sudo nft add rule inet nat postrouting oifname "eth1" ip saddr 192.168.100.0/24 counter masquerade
```



### Tráfico ssh saliente cortafuegos -> LAN

```
sudo nft add rule inet filter output oifname "eth2" ip daddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
sudo nft add rule inet filter input iifname "eth2" ip saddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
```

Pruebo que funciona:
```
vagrant@perimetral:~$ ssh vagrant@192.168.100.3
Linux local 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Feb  2 09:05:39 2022 from 192.168.121.1
vagrant@local:~$
```



### Tráfico loopback

```
sudo nft add rule inet filter output oifname "lo" counter accept
sudo nft add rule inet filter input iifname "lo" counter accept
```

Ya funciona el ping a localhost:
```
vagrant@perimetral:~$ ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.102 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.033 ms
^C
--- 127.0.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1014ms
rtt min/avg/max/mdev = 0.033/0.067/0.102/0.034 ms
```



### Tráfico ICMP

Permito ping entrante desde la red externa:
```
sudo nft add rule inet filter input iifname "eth1" icmp type echo-request counter accept
sudo nft add rule inet filter output oifname "eth1" icmp type echo-reply counter accept
```

Pruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej2-nftables-perimetral$ ping 172.22.7.193
PING 172.22.7.193 (172.22.7.193) 56(84) bytes of data.
64 bytes from 172.22.7.193: icmp_seq=1 ttl=64 time=0.278 ms
64 bytes from 172.22.7.193: icmp_seq=2 ttl=64 time=0.232 ms
^C
--- 172.22.7.193 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1028ms
rtt min/avg/max/mdev = 0.232/0.255/0.278/0.023 ms
```

Permito ping saliente cortafuegos -> LAN:
```
sudo nft add rule inet filter output oifname "eth2" icmp type echo-request counter accept
sudo nft add rule inet filter input iifname "eth2" icmp type echo-reply counter accept
```

Pruebo que funciona:
```
vagrant@perimetral:~$ ping 192.168.100.3
PING 192.168.100.3 (192.168.100.3) 56(84) bytes of data.
64 bytes from 192.168.100.3: icmp_seq=1 ttl=64 time=0.398 ms
64 bytes from 192.168.100.3: icmp_seq=2 ttl=64 time=0.325 ms
^C
--- 192.168.100.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1031ms
rtt min/avg/max/mdev = 0.325/0.361/0.398/0.036 ms
```



### FORWARD: tráfico icmp saliente LAN -> Internet

```
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip saddr 192.168.100.0/24 icmp type echo-request counter accept
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip daddr 192.168.100.0/24 icmp type echo-reply counter accept
```

Pruebo que funciona el ping a Internet:
```
vagrant@local:~$ ping 1.1.1.1 -c 2
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=438 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=171 ms

--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 170.639/304.483/438.327/133.844 ms
```



### FORWARD: tráfico DNS saliente desde LAN

```
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip saddr 192.168.100.0/24 udp dport 53 ct state new,established counter accept
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip daddr 192.168.100.0/24 udp sport 53 ct state established counter accept
```

Pruebo que funciona:
```
vagrant@local:~$ dig @1.1.1.1 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @1.1.1.1 www.example.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7637
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1


;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.example.org.	        IN	A

;; ANSWER SECTION:
www.example.org.	86400	IN	A	93.184.216.34

;; Query time: 516 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Wed Feb 02 12:00:55 UTC 2022
;; MSG SIZE  rcvd: 60
```



### FORWARD: tráfico http/https saliente desde LAN

```
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip protocol tcp ip saddr 192.168.100.0/24 tcp dport { 80,443 } ct state new,established counter accept
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip protocol tcp ip daddr 192.168.100.0/24 tcp sport { 80,443 } ct state established counter accept
```

Compruebo que funciona http:
```
vagrant@local:~$ curl portquiz.net:80
Port test successful!
Your IP: 158.99.1.24
```

Compruebo que funciona https:
```
vagrant@local:~$ curl portquiz.net:443
Port test successful!
Your IP: 158.99.1.24
```



### FORWARD: tráfico http entrante hacia LAN

```
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip daddr 192.168.100.0/24 tcp dport 80 ct state new,established counter accept
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip saddr 192.168.100.0/24 tcp sport 80 ct state established counter accept
```

Necesito una regla DNAT:
```
sudo nft add rule inet nat prerouting iifname "eth1" tcp dport 80 counter dnat ip to 192.168.100.3
```

Compruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej2-nftables-perimetral$ telnet 172.22.7.193 80
Trying 172.22.7.193...
Connected to 172.22.7.193.
Escape character is '^]'.

```






## Ejercicios

### Ejercicio 0
> Guardar la configuración del cortafuegos de forma persistente

```
nft list ruleset > /etc/nftables.conf
sudo systemctl enable nftables
sudo systemctl start nftables
```



### Ejercicio 1
> Permitir tráfico ssh saliente al exterior desde la máquina cortafuegos

```
sudo nft add rule inet filter output oifname "eth1" tcp dport 22 ct state new,established counter accept
sudo nft add rule inet filter input iifname "eth1" tcp sport 22 ct state established counter accept
```

Pruebo que funciona conectando a mi host:
```
vagrant@perimetral:~$ ssh atlas@192.168.1.106
atlas@192.168.1.106's password:
Linux olympus 5.10.0-8-amd64 #1 SMP Debian 5.10.46-4 (2021-08-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Wed Feb  2 22:45:29 2022 from 192.168.1.114
atlas@olympus:~$
```



### Ejercicio 2
> Permitir tráfico DNS saliente desde el cortafuegos sólo a 192.168.202.2. Comprobar que no se puede hacer un dig @1.1.1.1

```
sudo nft add rule inet filter output ip daddr 192.168.202.2 udp dport 53 ct state new,established counter accept
sudo nft add rule inet filter input ip saddr 192.168.202.2 udp sport 53 ct state established counter accept
```

Pruebo que las consultas DNS a 192.168.202.2 funcionan:
```
vagrant@perimetral:~$ dig @192.168.202.2 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @192.168.202.2 www.example.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39297
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 7eeeb71280505e6e78cad2c561fa79dd3cddfb164ad3691c (good)
;; QUESTION SECTION:
;www.example.org.	        IN	A

;; ANSWER SECTION:
www.example.org.	15752	IN	A	93.184.216.34

;; Query time: 0 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Wed Feb 02 12:32:29 UTC 2022
;; MSG SIZE  rcvd: 88
```

Pruebo que las consultas DNS a 1.1.1.1 no funcionan:
```
vagrant@perimetral:~$ dig @1.1.1.1 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @1.1.1.1 www.example.org
; (1 server found)
;; global options: +cmd
;; connection timed out; no servers could be reached
```



### Ejercicio 3
> Permitir que el cortafuegos pueda navegar por Internet

```
sudo nft add rule inet filter output oifname "eth1" ip protocol tcp tcp dport { 80,443 } ct state new,established counter accept
sudo nft add rule inet filter input iifname "eth1" ip protocol tcp tcp sport { 80,443 } ct state established counter accept
```

Compruebo que funciona http:
```
vagrant@perimetral:~$ telnet 52.47.209.216 80
Trying 52.47.209.216...
Connected to 52.47.209.216.
Escape character is '^]'.

```

Compruebo que funciona https:
```
vagrant@perimetral:~$ telnet 52.47.209.216 443
Trying 52.47.209.216...
Connected to 52.47.209.216.
Escape character is '^]'.

```



### Ejercicio 4
> Los equipos de la red local deben poder tener conexión al exterior

Este paso ya se hizo en las preliminares así que no lo repito. Se necesitan 2 cosas:

- SNAT
- Reglas forward para el tráfico ICMP saliente

Resumen de comandos para que funcione:
```
sudo nft add rule inet nat postrouting oifname "eth1" ip saddr 192.168.100.0/24 counter masquerade
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip saddr 192.168.100.0/24 icmp type echo-request counter accept
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip daddr 192.168.100.0/24 icmp type echo-reply counter accept
```



### Ejercicio 5
> Tráfico ssh saliente cortafuegos -> LAN

Este paso ya se hizo en las preliminares así que no lo repito. Se necesitan estas reglas:
```
sudo nft add rule inet filter output oifname "eth2" ip daddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
sudo nft add rule inet filter input iifname "eth2" ip saddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
```



### Ejercicio 6
> Permitir ping entrante LAN -> cortafuegos:

```
sudo nft add rule inet filter input iifname "eth2" icmp type echo-request counter accept
sudo nft add rule inet filter output oifname "eth2" icmp type echo-reply counter accept
```

Pruebo que funciona:
```
vagrant@local:~$ ping 192.168.100.2
PING 192.168.100.2 (192.168.100.2) 56(84) bytes of data.
64 bytes from 192.168.100.2: icmp_seq=1 ttl=64 time=0.312 ms
64 bytes from 192.168.100.2: icmp_seq=2 ttl=64 time=0.539 ms
^C
--- 192.168.100.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.312/0.425/0.539/0.113 ms
```



### Ejercicio 7
> Permitir tráfico ssh al exterior desde la LAN

```
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip saddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip daddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
```

Pruebo que funciona conectando a mi host para que atraviese el cortafuegos:
```
vagrant@local:~$ ssh atlas@192.168.1.106
atlas@192.168.1.106's password:
Linux olympus 5.10.0-8-amd64 #1 SMP Debian 5.10.46-4 (2021-08-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Thu Feb  3 01:31:42 2022 from 192.168.1.114
atlas@olympus:~$
```



### Ejercicio 8
> Permitir acceso desde el exterior al servidor de correos de la LAN

```
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip daddr 192.168.100.0/24 tcp dport 25 counter accept
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip saddr 192.168.100.0/24 tcp sport 25 counter accept
sudo nft add rule inet nat prerouting iifname "eth1" tcp dport 25 counter dnat ip to 192.168.100.3
```

Pruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej2-nftables-perimetral$ telnet 172.22.7.193 25
Trying 172.22.7.193...
Connected to 172.22.7.193.
Escape character is '^]'.
220 local ESMTP Postfix (Debian/GNU)

```

> Permitir acceso desde el cortafuegos al servidor de correos de la LAN

```
sudo nft add rule inet filter output ip daddr 192.168.100.3 tcp dport 25 counter accept
sudo nft add rule inet filter input ip saddr 192.168.100.3 tcp sport 25 counter accept
```

Pruebo que funciona desde el cortafuegos:
```
vagrant@perimetral:~$ telnet 192.168.100.3 25
Trying 192.168.100.3...
Connected to 192.168.100.3.
Escape character is '^]'.
220 local ESMTP Postfix (Debian/GNU)

```



### Ejercicio 9
> Permitir tráfico ssh desde el exterior a la LAN

```
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip daddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip saddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
sudo nft add rule inet nat prerouting iifname "eth1" tcp dport 22 counter dnat ip to 192.168.100.3
```

Pruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej2-nftables-perimetral$ ssh vagrant@172.22.7.193
Linux local 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  3 07:37:45 2022 from 192.168.100.2
vagrant@local:~$
```



### Ejercicio 10
> Modifica la regla anterior, para que al acceder desde el exterior por ssh tengamos que conectar al puerto 2222, aunque el servidor ssh este configurado para acceder por el puerto 22

```
sudo nft add rule inet nat prerouting iifname "eth1" tcp dport 2222 counter dnat ip to 192.168.100.3:22
```

Pruebo que funciona:
```
atlas@olympus:~/vagrant/ej2-nftables-perimetral$ ssh -p 2222 vagrant@172.22.7.193
Linux local 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  3 08:06:47 2022 from 172.22.9.227
vagrant@local:~$
```



### Ejercicio 11
> Permitir consultas DNS desde la LAN sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1

```
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip saddr 192.168.100.0/24 ip daddr 192.168.202.2 udp dport 53 ct state new,established counter accept
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip saddr 192.168.202.2 ip daddr 192.168.100.0/24 udp sport 53 ct state established counter accept
```

Pruebo que las consultas DNS a 192.168.202.2 funcionan:
```
vagrant@local:~$ dig @192.168.202.2 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @192.168.202.2 www.example.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45511
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 5

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: e8ba7a0a4e288162253fa85561fb919daab6c9aee27df0e8 (good)
;; QUESTION SECTION:
;www.example.org.	        IN	A

;; ANSWER SECTION:
www.example.org.	83378	IN	A	93.184.216.34

;; AUTHORITY SECTION:
example.org.	        82302	IN	NS	b.iana-servers.net.
example.org.	        82302	IN	NS	a.iana-servers.net.

;; ADDITIONAL SECTION:
a.iana-servers.net.	82275	IN	A	199.43.135.53
b.iana-servers.net.	82275	IN	A	199.43.133.53
a.iana-servers.net.	82275	IN	AAAA	2001:500:8f::53
b.iana-servers.net.	82275	IN	AAAA	2001:500:8d::53

;; Query time: 0 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Thu Feb 03 08:26:05 UTC 2022
;; MSG SIZE  rcvd: 224
```

Pruebo que las consultas DNS a 1.1.1.1 no funcionan:
```
vagrant@local:~$ dig @1.1.1.1 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @1.1.1.1 www.example.org
; (1 server found)
;; global options: +cmd
;; connection timed out; no servers could be reached
```



### Ejercicio 12
> Permite que los equipos de la LAN puedan navegar por Internet

Este paso ya se hizo en las preliminares así que no lo repito. Se necesitan estas reglas:
```
sudo nft add rule inet filter forward iifname "eth2" oifname "eth1" ip protocol tcp ip saddr 192.168.100.0/24 tcp dport { 80,443 } ct state new,established counter accept
sudo nft add rule inet filter forward iifname "eth1" oifname "eth2" ip protocol tcp ip daddr 192.168.100.0/24 tcp sport { 80,443 } ct state established counter accept
```
