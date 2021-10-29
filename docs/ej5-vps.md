# Ejercicio 5: Contratación y configuración de un VPS


## ENTREGA

> Nombre completo de la máquina, para que se pueda acceder con el usuario profesor

```
kampe.adrianjaramillo.tk
```



## CONFIGURACIÓN DEL VPS

### Paso 1

> Modificar hostname

Editamos el fichero `/etc/hostname`:
```
root@localhost:~# cat /etc/hostname
kampe
```

Este cambio tomará lugar al reiniciar, pero si no queremos hacerlo, ejecutamos:
```
sudo hostname kampe
```

Este comando es temporal, lo que hace el cambio permanente realmente es haber editado previamente el fichero `/etc/hostname`.

Verificamos que el cambio ha surtido efecto:
```
root@localhost:~# hostname
kampe
```

El hostname ha cambiado pero nuestro prompt no. Esto se debe a que la shell en la que nos encontramos se encuentra "desactualizada" por así decirlo.

Para arreglar esto, simplemente abrimos una nueva shell y ya deberíamos de tener nuestro hostname actualizado en el prompt:
```
root@localhost:~# bash
root@kampe:~#
```

> Cambiar FQDN de la máquina a `kampe.adrianjaramillo.tk`

Modifico `/etc/hosts` de la siguiente manera:
```
127.0.0.1       localhost.localdomain localhost

87.106.228.149 kampe.adrianjaramillo.tk kampe

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

> Comprobar el funcionamiento haciendo `hostname -f`

```
root@kampe:~# hostname -f
kampe.adrianjaramillo.tk
```

> Compruebo el nombre corto

```
blackmamba@kampe:~$ ping kampe
PING kampe.adrianjaramillo.tk (87.106.228.149) 56(84) bytes of data.
64 bytes from kampe.adrianjaramillo.tk (87.106.228.149): icmp_seq=1 ttl=64 time=0.021 ms
64 bytes from kampe.adrianjaramillo.tk (87.106.228.149): icmp_seq=2 ttl=64 time=0.040 ms
^C
--- kampe.adrianjaramillo.tk ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1010ms
rtt min/avg/max/mdev = 0.021/0.030/0.040/0.009 ms
```


### Paso 2
> Crear usuario normal, con contraseña robusta

```
adduser blackmamba
```

Su password es: `?UdLbQ?*3'd6)@C3`

> Habilitar sudo para `blackmamba`

```
usermod -aG sudo blackmamba
```

> Habilitar sudo sin contraseña para `blackmamba`

Añadir lo siguiente en `/etc/sudoers`:
```
blackmamba     ALL=(ALL) NOPASSWD:ALL
```

Importante que se añada **al final del fichero**.


> Pasar mi clave pública SSH a `blackmamba`

```
ssh-copy-id blackmamba@kampe.adrianjaramillo.tk
```

> Cambiar los siguientes parámetros en `/etc/ssh/sshd_config`

```
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
PermitRootLogin no
PermitRootLogin prohibit-password
```

> Reiniciar ssh

```
sudo systemctl restart ssh
```

> Comprobar que no se puede acceder como root

```
atlas@olympus:~$ ssh root@kampe.adrianjaramillo.tk
root@kampe.adrianjaramillo.tk: Permission denied (publickey).
```



### Paso 3
> Crear registro DNS tipo A con el nombre de la máquina y la IP pública asociada

![](https://i.imgur.com/bUB1QTG.png)

> Comprobar que la resolución funciona

```
dig kampe.adrianjaramillo.tk
```
```
; <<>> DiG 9.16.15-Debian <<>> kampe.adrianjaramillo.tk
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43729
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 9

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 5a78a81a29a907cc4f5f5213617bcc35d6b26dd93853c9e2 (good)
;; QUESTION SECTION:
;kampe.adrianjaramillo.tk.	IN	A

;; ANSWER SECTION:
kampe.adrianjaramillo.tk. 3083	IN	A	87.106.228.149

;; AUTHORITY SECTION:
tk.			172283	IN	NS	b.ns.tk.
tk.			172283	IN	NS	a.ns.tk.
tk.			172283	IN	NS	c.ns.tk.
tk.			172283	IN	NS	d.ns.tk.

;; ADDITIONAL SECTION:
a.ns.tk.		172283	IN	A	194.0.38.1
b.ns.tk.		172283	IN	A	194.0.39.1
c.ns.tk.		172283	IN	A	194.0.40.1
d.ns.tk.		172283	IN	A	194.0.41.1
a.ns.tk.		172283	IN	AAAA	2001:678:50::1
b.ns.tk.		172283	IN	AAAA	2001:678:54::1
c.ns.tk.		172283	IN	AAAA	2001:678:58::1
d.ns.tk.		172283	IN	AAAA	2001:678:5c::1

;; Query time: 4 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Fri Oct 29 12:25:57 CEST 2021
;; MSG SIZE  rcvd: 340
```



### Paso 4
> Crear usuario profesor, con contraseña robusta

```
sudo adduser profesor
```

Su password es: `?UdLbQ?*3'd6)@C4`

> Habilitar sudo para `profesor`

```
sudo usermod -aG sudo profesor
```

> Habilitar sudo sin contraseña para `profesor`

Añadir lo siguiente en `/etc/sudoers`:
```
profesor     ALL=(ALL) NOPASSWD:ALL
```

Importante que se añada **al final del fichero**

> Añadir clave pública SSH de josedom a `profesor`

Tras descargar su clave, la paso a mi VPS:
```
scp Downloads/josedom-ssh.pub blackmamba@kampe.adrianjaramillo.tk:/home/blackmamba
```

Añado esa clave ssh pública al usuario profesor:
```
mkdir /home/profesor/.ssh && touch /home/profesor/.ssh/authorized_keys && cat /home/blackmamba/josedom-ssh.pub > /home/profesor/.ssh/authorized_keys
```
