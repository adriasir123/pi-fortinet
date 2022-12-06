# Ejercicio 2 iptables: cortafuegos perimetral



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
      :libvirt__network_name => "ej2-iptables-perimetral",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.100.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :local do |local|
    local.vm.box = "debian/bullseye64"
    local.vm.hostname = "local"
    local.vm.network :private_network,
      :libvirt__network_name => "ej2-iptables-perimetral",
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
sudo ip route add default via 192.168.1.1
```

Cambio la ruta por defecto de la máquina de la LAN:
```
sudo ip route del default
sudo ip route add default via 192.168.100.2
```

Instalo un servidor web y un servidor de correos en la máquina de la LAN para dejar abierto el puerto 80 y el 25:
```
sudo apt update
sudo apt install apache2
sudo apt install postfix
```



### Tráfico ssh entrante al cortafuegos

```
sudo iptables -A INPUT -s 192.168.121.0/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -d 192.168.121.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```



### Políticas por defecto

```
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP
```

No puedo hacer ping a localhost:
```
vagrant@perimetral:~$ ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
^C
--- 127.0.0.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2052ms
```

No puedo hacer ping a Internet:
```
vagrant@perimetral:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
^C
--- 1.1.1.1 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3077ms
```

Si no se nos corta la conexión ssh a la máquina después de este paso, el par de reglas que aplicamos antes funcionan.



### Activar bit de forward

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```



### SNAT

```
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth1 -j MASQUERADE
```



### Tráfico ssh saliente cortafuegos -> LAN

```
sudo iptables -A OUTPUT -p tcp -o eth2 -d 192.168.100.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp -i eth2 -s 192.168.100.0/24 --sport 22 -m state --state ESTABLISHED -j ACCEPT
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
Last login: Mon Jan 31 08:25:49 2022 from 192.168.100.2
vagrant@local:~$
```



### Tráfico loopback

```
sudo iptables -A OUTPUT -o lo -p icmp -j ACCEPT
sudo iptables -A INPUT -i lo -p icmp -j ACCEPT
```

Ya funciona el ping a localhost:
```
vagrant@perimetral:~$ ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.023 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.076 ms
^C
--- 127.0.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1005ms
rtt min/avg/max/mdev = 0.023/0.049/0.076/0.026 ms
```



### Tráfico ICMP

Permito ping entrante desde la red externa:
```
sudo iptables -A INPUT -i eth1 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A OUTPUT -o eth1 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej2-iptables-perimetral$ ping 192.168.1.110
PING 192.168.1.110 (192.168.1.110) 56(84) bytes of data.
64 bytes from 192.168.1.110: icmp_seq=1 ttl=64 time=0.147 ms
64 bytes from 192.168.1.110: icmp_seq=2 ttl=64 time=0.301 ms
^C
--- 192.168.1.110 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1024ms
rtt min/avg/max/mdev = 0.147/0.224/0.301/0.077 ms
```

Permito ping saliente cortafuegos -> LAN:
```
sudo iptables -A OUTPUT -o eth2 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A INPUT -i eth2 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona:
```
vagrant@perimetral:~$ ping 192.168.100.3
PING 192.168.100.3 (192.168.100.3) 56(84) bytes of data.
64 bytes from 192.168.100.3: icmp_seq=1 ttl=64 time=0.268 ms
64 bytes from 192.168.100.3: icmp_seq=2 ttl=64 time=0.443 ms
^C
--- 192.168.100.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1009ms
rtt min/avg/max/mdev = 0.268/0.355/0.443/0.087 ms
```



### FORWARD: tráfico icmp saliente LAN -> Internet

```
sudo iptables -A FORWARD -o eth1 -i eth2 -s 192.168.100.0/24 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth2 -d 192.168.100.0/24 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona el ping a Internet:
```
vagrant@local:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=42.5 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=132 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 42.503/87.035/131.567/44.532 ms
```



### FORWARD: tráfico DNS saliente desde LAN

```
sudo iptables -A FORWARD -i eth2 -o eth1 -s 192.168.100.0/24 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -o eth2 -i eth1 -d 192.168.100.0/24 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona:
```
vagrant@local:~$ dig @1.1.1.1 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @1.1.1.1 www.example.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3057
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.example.org.	        IN	A

;; ANSWER SECTION:
www.example.org.	86400	IN	A	93.184.216.34

;; Query time: 508 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Mon Jan 31 11:34:11 UTC 2022
;; MSG SIZE  rcvd: 60
```



### FORWARD: tráfico http/https saliente desde LAN

```
sudo iptables -A FORWARD -i eth2 -o eth1 -s 192.168.100.0/24 -p tcp -m multiport --dport 80,443 -m state --state NEW,ESTABLISHED -m time --timestart 01:00 --timestop 22:00 -j ACCEPT
sudo iptables -A FORWARD -o eth2 -i eth1 -d 192.168.100.0/24 -p tcp -m multiport --sport 80,443 -m state --state ESTABLISHED -m time --timestart 01:00 --timestop 22:00 -j ACCEPT
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
sudo iptables -A FORWARD -i eth1 -o eth2 -d 192.168.100.0/24 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth2 -o eth1 -s 192.168.100.0/24 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

Necesito una regla DNAT:
```
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j DNAT --to 192.168.100.3
```

Compruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej2-iptables-perimetral$ telnet 192.168.1.110 80
Trying 192.168.1.110...
Connected to 192.168.1.110.
Escape character is '^]'.

```






## Ejercicios

### Ejercicio 0
> Guardar la configuración del cortafuegos de forma persistente

```
sudo apt install iptables-persistent
```
*(sólo con instalar este paquete es suficiente, ya que durante la instalación se nos pregunta que si queremos guardar las reglas y sólo tenemos que responder que sí)*



### Ejercicio 1
> Permitir tráfico ssh saliente desde la máquina cortafuegos

```
sudo iptables -A OUTPUT -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
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
Last login: Tue Feb  1 06:35:39 2022 from 192.168.1.110
atlas@olympus:~$
```



### Ejercicio 2
> Permitir tráfico DNS saliente desde el cortafuegos sólo a 192.168.202.2. Comprobar que no se puede hacer un dig @1.1.1.1

```
sudo iptables -A OUTPUT -d 192.168.202.2 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -s 192.168.202.2 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que las consultas DNS a 192.168.202.2 funcionan:
```
vagrant@perimetral:~$ dig @192.168.202.2 www.example.org

; <<>> DiG 9.16.15-Debian <<>> @192.168.202.2 www.example.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27464
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: ecefbd1dc86382b6f96bcd7a61fa34bcef09c617d031a22a (good)
;; QUESTION SECTION:
;www.example.org.	        IN	A

;; ANSWER SECTION:
www.example.org.	33449	IN	A	93.184.216.34

;; Query time: 0 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Wed Feb 02 07:37:32 UTC 2022
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
sudo iptables -A OUTPUT -o eth1 -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i eth1 -p tcp -m multiport --sports 80,443 -m state --state ESTABLISHED -j ACCEPT
```

Compruebo que funciona http:
```
vagrant@perimetral:~$ curl portquiz.net:80
Port test successful!
Your IP: 90.168.73.6
```

Compruebo que funciona https:
```
vagrant@perimetral:~$ curl portquiz.net:443
Port test successful!
Your IP: 90.168.73.6
```



### Ejercicio 4
> Los equipos de la red local deben poder tener conexión al exterior

Este paso ya se hizo en las preliminares así que no lo repito. Se necesitan 2 cosas:

- SNAT
- Reglas forward para el tráfico ICMP saliente

Resumen de comandos para que funcione:
```
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth1 -j MASQUERADE
sudo iptables -A FORWARD -o eth1 -i eth2 -s 192.168.100.0/24 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth2 -d 192.168.100.0/24 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```



### Ejercicio 5
> Tráfico ssh saliente cortafuegos -> LAN

Este paso ya se hizo en las preliminares así que no lo repito. Se necesitan estas reglas:
```
sudo iptables -A OUTPUT -p tcp -o eth2 -d 192.168.100.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp -i eth2 -s 192.168.100.0/24 --sport 22 -m state --state ESTABLISHED -j ACCEPT
```



### Ejercicio 6
> Permitir ping entrante LAN -> cortafuegos:

```
sudo iptables -A INPUT -i eth2 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A OUTPUT -o eth2 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona:
```
vagrant@local:~$ ping 192.168.100.2
PING 192.168.100.2 (192.168.100.2) 56(84) bytes of data.
64 bytes from 192.168.100.2: icmp_seq=1 ttl=64 time=0.355 ms
64 bytes from 192.168.100.2: icmp_seq=2 ttl=64 time=0.259 ms
^C
--- 192.168.100.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1010ms
rtt min/avg/max/mdev = 0.259/0.307/0.355/0.048 ms
```



### Ejercicio 7
> Permitir tráfico ssh al exterior desde la LAN

```
sudo iptables -A FORWARD -i eth2 -o eth1 -s 192.168.100.0/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth2 -d 192.168.100.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
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
Last login: Tue Feb  1 21:40:41 2022 from 192.168.1.113
atlas@olympus:~$
```



### Ejercicio 8
> Permitir acceso desde el exterior al servidor de correos de la LAN

```
sudo iptables -A FORWARD -i eth1 -o eth2 -d 192.168.100.0/24 -p tcp --dport 25 -j ACCEPT
sudo iptables -A FORWARD -i eth2 -o eth1 -s 192.168.100.0/24 -p tcp --sport 25 -j ACCEPT
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 25 -j DNAT --to 192.168.100.3
```

Pruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej2-iptables-perimetral$ telnet 192.168.1.113 25
Trying 192.168.1.113...
Connected to 192.168.1.113.
Escape character is '^]'.
220 local ESMTP Postfix (Debian/GNU)

```

> Permitir acceso desde el cortafuegos al servidor de correos de la LAN

```
sudo iptables -A OUTPUT -d 192.168.100.3 -p tcp --dport 25 -j ACCEPT
sudo iptables -A INPUT -s 192.168.100.3 -p tcp --sport 25 -j ACCEPT
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
sudo iptables -A FORWARD -i eth1 -o eth2 -d 192.168.100.0/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth2 -o eth1 -s 192.168.100.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 22 -j DNAT --to 192.168.100.3
```

Pruebo que funciona desde mi host:
```
atlas@olympus:~/vagrant/ej2-iptables-perimetral$ ssh vagrant@192.168.1.113
Linux local 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Feb  1 20:01:00 2022 from 192.168.100.2
vagrant@local:~$
```



### Ejercicio 10
> Modifica la regla anterior, para que al acceder desde el exterior por ssh tengamos que conectar al puerto 2222, aunque el servidor ssh este configurado para acceder por el puerto 22

```
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 2222 -j DNAT --to-destination 192.168.100.3:22
```

Pruebo que funciona:
```
atlas@olympus:~/vagrant/ej2-iptables-perimetral$ ssh -p 2222 vagrant@192.168.1.113
Linux local 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Feb  1 23:54:20 2022 from 192.168.1.106
vagrant@local:~$
```



### Ejercicio 11
> Permitir consultas DNS desde la LAN sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1

```
sudo iptables -A FORWARD -i eth2 -o eth1 -s 192.168.100.0/24 -d 192.168.202.2 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth2 -s 192.168.202.2 -d 192.168.100.0/24 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que las consultas DNS a 192.168.202.2 funcionan:
```
vagrant@local:~$ dig @192.168.202.2 www.example.org

; <<>> DiG 9.16.22-Debian <<>> @192.168.202.2 www.example.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21410
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: a69a6f7c799135d6042bcb5f61fa3a83c62614c5b25bc08b (good)
;; QUESTION SECTION:
;www.example.org.	        IN	A

;; ANSWER SECTION:
www.example.org.	31970	IN	A	93.184.216.34

;; Query time: 4 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Wed Feb 02 08:02:11 UTC 2022
;; MSG SIZE  rcvd: 88
```

Pruebo que las consultas DNS a 1.1.1.1 no funcionan:
```
vagrant@local:~$ dig @1.1.1.1 www.example.org

; <<>> DiG 9.16.22-Debian <<>> @1.1.1.1 www.example.org
; (1 server found)
;; global options: +cmd
;; connection timed out; no servers could be reached
```



### Ejercicio 12
> Permite que los equipos de la LAN puedan navegar por Internet

Este paso ya se hizo en las preliminares así que no lo repito. Se necesitan estas reglas:
```
sudo iptables -A FORWARD -i eth2 -o eth1 -s 192.168.100.0/24 -p tcp -m multiport --dport 80,443 -m state --state NEW,ESTABLISHED -m time --timestart 01:00 --timestop 22:00 -j ACCEPT
sudo iptables -A FORWARD -o eth2 -i eth1 -d 192.168.100.0/24 -p tcp -m multiport --sport 80,443 -m state --state ESTABLISHED -m time --timestart 01:00 --timestop 22:00 -j ACCEPT
```
