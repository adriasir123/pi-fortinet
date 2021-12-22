# Práctica: VPN

## Parte 1: acceso remoto OpenVPN

### Escenario

![](https://i.postimg.cc/tJktZ6nV/acceso-remoto-openvpn.jpg)

`Vagrantfile`:
```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 1
    libvirt.memory = 512
  end

  config.vm.define :clientevpn do |clientevpn|
    clientevpn.vm.box = "debian/bullseye64"
    clientevpn.vm.hostname = "clientevpn"
    clientevpn.vm.network :private_network,
      :libvirt__network_name => "externa-vpn",
      :libvirt__dhcp_enabled => false,
      :ip => "192.88.99.11",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servervpn do |servervpn|
    servervpn.vm.box = "debian/bullseye64"
    servervpn.vm.hostname = "servervpn"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "externa-vpn",
      :libvirt__dhcp_enabled => false,
      :ip => "192.88.99.10",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "interna-vpn",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientenormal do |clientenormal|
    clientenormal.vm.box = "debian/bullseye64"
    clientenormal.vm.hostname = "clientenormal"
    clientenormal.vm.network :private_network,
      :libvirt__network_name => "interna-vpn",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.5",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```



### En servervpn

#### Instalar OpenVPN

```
sudo apt update
sudo apt install openvpn
```

#### Habilitar forwarding

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### Crear infraestructura de clave pública

Según la guía que estoy siguiendo es necesario copiar la configuración de `/usr/share/easy-rsa` a `/etc/openvpn` antes de continuar, para prevenir que futuras actualizaciones del paquete OpenVPN modifiquen las configuraciones que hagamos.

En nuestro caso esta posibilidad **nunca llegaría a hacerse realidad**, puesto que lo que estamos haciendo es simplemente una práctica, que se extiende durante un periodo de tiempo relativamente corto.

Aún así, lo hago por razones de organización, ya que todas las configuraciones manuales deben estar bajo `/etc`.

```
sudo cp -r /usr/share/easy-rsa /etc/openvpn
cd /etc/openvpn/easy-rsa
```

Inicializamos la PKI:
```
sudo ./easyrsa init-pki
```
```
init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa/pki
```

#### Generar certificado + clave privada de la CA

En este paso obtendremos el par de claves necesario de la Autoridad Certificadora (CA).

Posteriormente, es esta clave privada la que se usará para firmar los certificados tanto del servidor como del cliente.

```
sudo ./easyrsa build-ca
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021

Enter New CA Key Passphrase:
Re-Enter New CA Key Passphrase:
Generating RSA private key, 2048 bit long modulus (2 primes)
........................................................................................+++++
.......................................+++++
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:Adrianj CA

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/easy-rsa/pki/ca.crt
```
Passphrase: 1234

Certificado generado: `/etc/openvpn/easy-rsa/pki/ca.crt`  
Clave privada generada: `/etc/openvpn/easy-rsa/pki/private/ca.key`

#### Generar certificado + clave privada del servidor VPN

```
sudo ./easyrsa build-server-full server nopass
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021
Generating a RSA private key
.......+++++
............................+++++
writing new private key to '/etc/openvpn/easy-rsa/pki/easy-rsa-2389.Tqf1G1/tmp.Ow20MK'
-----
Using configuration from /etc/openvpn/easy-rsa/pki/easy-rsa-2389.Tqf1G1/tmp.5Exl9Q
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Mar 18 08:30:09 2024 GMT (825 days)

Write out database with 1 new entries
Data Base Updated
```

Certificado generado: `/etc/openvpn/easy-rsa/pki/issued/server.crt`  
Clave privada generada: `/etc/openvpn/easy-rsa/pki/private/server.key`

#### Generar parámetros Diffie–Hellman

```
sudo ./easyrsa gen-dh
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
.......................+.....................+..............................................................+......................+................................................................................+...................................................................................................................................................+....................+....................................................................+..................+.......+..............................................................+...........................................................................+................................................................................................................................+.....................................+...........................................................................................................................................................................+................................................................................................................................................................................................................+.+.............................+.........................+..............................................................................................................................................................................................................................+..............+......................................................................+................................+.................................+......................................................................................................+............................................................................................+...........................................................................................................................................................................................+........................................+.............................................................................+....................................................................................................+...........+...........................................................+.....................................................................+...............................................................................................................................................+...........................................................................................................................................+.....................................+........................................................................................+...................+.............+.......................+.......................................................................................+.........................................................+...........................................................................+...............................................................................................................................................................+.......................................+........................................................+..........................................+................................................................................................................+.........+.............................+..........................................................................+...............................................+............................................................................................................+.......+...............................................................................................................+............................................+................+..................................................+...............+..............................................................................+.......................................................................+.................................+..+.+.+..............................................................................................+.............................................................................+...................................................................................................................................................................................................+...........+................................................................................+............................................................................................................................................................................................................+............................+..+......................................................................................................................................................................................+..............................................................................................+......................................................................................................................................................................................................................................................................................................................................................................................................+........................................+......................+...............................+...........................+...............................................................................................+..........................................................................................................................+.......................................................................+.................................................................................+....................................................................+................................+.++*++*++*++*

DH parameters of size 2048 created at /etc/openvpn/easy-rsa/pki/dh.pem
```

#### Generar certificado + clave privada del cliente VPN

```
sudo ./easyrsa build-client-full clientevpn nopass
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021
Generating a RSA private key
..............+++++
................................+++++
writing new private key to '/etc/openvpn/easy-rsa/pki/easy-rsa-2541.yaiUd0/tmp.WePx19'
-----
Using configuration from /etc/openvpn/easy-rsa/pki/easy-rsa-2541.yaiUd0/tmp.GMaG7k
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'clientevpn'
Certificate is to be certified until Mar 18 08:49:43 2024 GMT (825 days)

Write out database with 1 new entries
Data Base Updated
```

Certificado generado: `/etc/openvpn/easy-rsa/pki/issued/clientevpn.crt`  
Clave privada generada: `/etc/openvpn/easy-rsa/pki/private/clientevpn.key`

#### Pasar ficheros necesarios al cliente VPN

```
sudo cp -rp /etc/openvpn/easy-rsa/pki/{ca.crt,issued/clientevpn.crt,private/clientevpn.key} ~
sudo chown vagrant: ~/{ca.crt,clientevpn.crt,clientevpn.key}
scp ~/{ca.crt,clientevpn.crt,clientevpn.key} vagrant@192.88.99.11:/home/vagrant
```

#### Configurar el servidor VPN

Copio la plantilla oficial de configuración para servidores openvpn, para partir de ella:
```
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/servidor.conf
```

Modifico `/etc/openvpn/server/servidor.conf`:
```
port 1194
proto udp
dev tun


ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem


topology subnet


# DIRECCIONAMIENTO PARA EL TÚNEL
#
# El servidor será: 10.99.99.1
# El resto de IPs estarán disponibles para los clientes
server 10.99.99.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt


push "route 192.168.0.0 255.255.255.0"


keepalive 10 120
cipher AES-256-CBC
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
```

#### Iniciar el servidor

```
sudo systemctl enable --now openvpn-server@servidor
```

Verifico que está funcionando:
```
sudo systemctl status openvpn-server@servidor
```
```
● openvpn-server@servidor.service - OpenVPN service for servidor
     Loaded: loaded (/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-12-14 09:16:30 UTC; 35s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 2800 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 528)
     Memory: 1.0M
        CPU: 9ms
     CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@servidor.service
             └─2800 /usr/sbin/openvpn --status /run/openvpn-server/status-servidor.log --status-version 2 --suppress-timestamps --config servidor.conf

Dec 14 09:16:30 servervpn openvpn[2800]: net_iface_up: set tun0 up
Dec 14 09:16:30 servervpn openvpn[2800]: net_addr_v4_add: 10.99.99.1/24 dev tun0
Dec 14 09:16:30 servervpn openvpn[2800]: Could not determine IPv4/IPv6 protocol. Using AF_INET
Dec 14 09:16:30 servervpn openvpn[2800]: Socket Buffers: R=[212992->212992] S=[212992->212992]
Dec 14 09:16:30 servervpn openvpn[2800]: UDPv4 link local (bound): [AF_INET][undef]:1194
Dec 14 09:16:30 servervpn openvpn[2800]: UDPv4 link remote: [AF_UNSPEC]
Dec 14 09:16:30 servervpn openvpn[2800]: MULTI: multi_init called, r=256 v=256
Dec 14 09:16:30 servervpn openvpn[2800]: IFCONFIG POOL IPv4: base=10.99.99.2 size=252
Dec 14 09:16:30 servervpn openvpn[2800]: IFCONFIG POOL LIST
Dec 14 09:16:30 servervpn openvpn[2800]: Initialization Sequence Completed
```



### En clientevpn

#### Instalar OpenVPN

```
sudo apt update
sudo apt install openvpn
```

#### Mover ficheros a la ruta adecuada

```
sudo mv ~/{ca.crt,clientevpn.crt,clientevpn.key} /etc/openvpn/client
sudo chown root: /etc/openvpn/client/*
```

`/etc/openvpn/client` queda así:
```
vagrant@clientevpn:/etc/openvpn/client$ ls -la
total 24
drwxr-xr-x 2 root root 4096 Dec 14 09:31 .
drwxr-xr-x 4 root root 4096 Dec 14 09:30 ..
-rw------- 1 root root 1200 Dec 14 09:01 ca.crt
-rw------- 1 root root 4498 Dec 14 09:01 clientevpn.crt
-rw------- 1 root root 1704 Dec 14 09:01 clientevpn.key
```

#### Configurar el cliente VPN

Copio la plantilla oficial de configuración para clientes openvpn, para partir de ella:
```
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client/cliente.conf
```

Modifico `/etc/openvpn/client/cliente.conf`:
```
client
dev tun
proto udp

remote 192.88.99.10 1194
resolv-retry infinite
nobind

persist-key
persist-tun

ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/clientevpn.crt
key /etc/openvpn/client/clientevpn.key

remote-cert-tls server
cipher AES-256-CBC
verb 3
```

Habilito el servicio:
```
sudo systemctl enable --now openvpn-client@cliente
```

Verifico que funciona:
```
sudo systemctl status openvpn-client@cliente
```
```
● openvpn-client@cliente.service - OpenVPN tunnel for cliente
     Loaded: loaded (/lib/systemd/system/openvpn-client@.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-12-14 09:43:44 UTC; 5s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 12859 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 528)
     Memory: 1.9M
        CPU: 17ms
     CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@cliente.service
             └─12859 /usr/sbin/openvpn --suppress-timestamps --nobind --config cliente.conf

Dec 14 09:43:44 clientevpn openvpn[12859]: net_route_v4_best_gw query: dst 0.0.0.0
Dec 14 09:43:44 clientevpn openvpn[12859]: net_route_v4_best_gw result: via 192.168.121.1 dev eth0
Dec 14 09:43:44 clientevpn openvpn[12859]: ROUTE_GATEWAY 192.168.121.1/255.255.255.0 IFACE=eth0 HWADD>
Dec 14 09:43:44 clientevpn openvpn[12859]: TUN/TAP device tun0 opened
Dec 14 09:43:44 clientevpn openvpn[12859]: net_iface_mtu_set: mtu 1500 for tun0
Dec 14 09:43:44 clientevpn openvpn[12859]: net_iface_up: set tun0 up
Dec 14 09:43:44 clientevpn openvpn[12859]: net_addr_v4_add: 10.99.99.2/24 dev tun0
Dec 14 09:43:44 clientevpn openvpn[12859]: net_route_v4_add: 192.168.0.0/24 via 10.99.99.1 dev [NULL]>
Dec 14 09:43:44 clientevpn openvpn[12859]: WARNING: this configuration may cache passwords in memory >
Dec 14 09:43:44 clientevpn openvpn[12859]: Initialization Sequence Completed
```



### En clientenormal

#### Cambiar ruta por defecto

```
sudo ip route del default
sudo ip route add default via 192.168.0.1
```



### Pruebas de funcionamiento

*(todas estarán hechas desde clientevpn)*

Ping al otro extremo del túnel *(servervpn)*:
```
vagrant@clientevpn:/etc/openvpn/client$ ping 10.99.99.1
PING 10.99.99.1 (10.99.99.1) 56(84) bytes of data.
64 bytes from 10.99.99.1: icmp_seq=1 ttl=64 time=0.412 ms
64 bytes from 10.99.99.1: icmp_seq=2 ttl=64 time=0.696 ms
64 bytes from 10.99.99.1: icmp_seq=3 ttl=64 time=0.455 ms
^C
--- 10.99.99.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2048ms
rtt min/avg/max/mdev = 0.412/0.521/0.696/0.124 ms
```

Ping a clientenormal:
```
vagrant@clientevpn:/etc/openvpn/client$ ping 192.168.0.5
PING 192.168.0.5 (192.168.0.5) 56(84) bytes of data.
64 bytes from 192.168.0.5: icmp_seq=1 ttl=63 time=0.849 ms
64 bytes from 192.168.0.5: icmp_seq=2 ttl=63 time=0.839 ms
^C
--- 192.168.0.5 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.839/0.844/0.849/0.005 ms
```

Traceroute a clientenormal *(para que se vea claro la utilización del túnel VPN)*:
```
vagrant@clientevpn:/etc/openvpn/client$ traceroute 192.168.0.5
traceroute to 192.168.0.5 (192.168.0.5), 30 hops max, 60 byte packets
 1  10.99.99.1 (10.99.99.1)  0.446 ms  0.364 ms  0.331 ms
 2  192.168.0.5 (192.168.0.5)  0.553 ms  0.612 ms  0.581 ms
```






## Parte 2: sitio a sitio OpenVPN

### Escenario

![]()

`Vagrantfile`:
```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 1
    libvirt.memory = 512
  end

  config.vm.define :clientenormal1 do |clientenormal1|
    clientenormal1.vm.box = "debian/bullseye64"
    clientenormal1.vm.hostname = "clientenormal1"
    clientenormal1.vm.network :private_network,
      :libvirt__network_name => "privada1-vpn2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientevpn do |clientevpn|
    clientevpn.vm.box = "debian/bullseye64"
    clientevpn.vm.hostname = "clientevpn"
    clientevpn.vm.network :private_network,
      :libvirt__network_name => "privada1-vpn2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    clientevpn.vm.network :private_network,
      :libvirt__network_name => "internet-vpn2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.88.99.11",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servervpn do |servervpn|
    servervpn.vm.box = "debian/bullseye64"
    servervpn.vm.hostname = "servervpn"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "internet-vpn2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.88.99.10",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "privada2-vpn2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.1.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientenormal2 do |clientenormal2|
    clientenormal2.vm.box = "debian/bullseye64"
    clientenormal2.vm.hostname = "clientenormal2"
    clientenormal2.vm.network :private_network,
      :libvirt__network_name => "privada2-vpn2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.1.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```



### En servervpn

#### Instalar OpenVPN

```
sudo apt update
sudo apt install openvpn
```

#### Habilitar forwarding

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### Crear PKI

```
sudo cp -r /usr/share/easy-rsa /etc/openvpn
cd /etc/openvpn/easy-rsa
sudo ./easyrsa init-pki
```
```
init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa/pki
```

#### Generar certificado + clave privada de la CA

```
sudo ./easyrsa build-ca
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021

Enter New CA Key Passphrase:
Re-Enter New CA Key Passphrase:
Generating RSA private key, 2048 bit long modulus (2 primes)
..............................................+++++
...............+++++
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:Adrianj CA    

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/easy-rsa/pki/ca.crt
```
Passphrase: 1234

#### Generar certificado + clave privada del servidor VPN

```
sudo ./easyrsa build-server-full server nopass
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021
Generating a RSA private key
...................................................................................................+++++
..................................+++++
writing new private key to '/etc/openvpn/easy-rsa/pki/easy-rsa-10904.7kk1nQ/tmp.Ek6j6Z'
-----
Using configuration from /etc/openvpn/easy-rsa/pki/easy-rsa-10904.7kk1nQ/tmp.uKADZb
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Mar 24 09:21:16 2024 GMT (825 days)

Write out database with 1 new entries
Data Base Updated
```

#### Generar parámetros Diffie–Hellman

```
sudo ./easyrsa gen-dh
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
............................................................................................+.....................................................................+.............................................................................................................................................................+.......................................................................+....................................+............................................................................................+.............................................+................................+.......................................................................................+..............................+.....................................................+..................................................................................................................+............................................................................+.....+...........+................................................................................................................................................................................................................................................................................................................................................+..............+...................+...............................................................................................+.............................................................+.......................................+......................................................................................................................+.+..............+...............................................................+............+...................+.....................................+............................................................+......................................+.................................................................................................................................................................................................................+......................................................................+.................................................................................................+...........................................................................................................................................................................+..............................................+.................................................................................................................................................+.....................................................+.............................+.....................................................................................................................+.........................................................................+............................................................................................................................................................+......................................................................................................................................+......................................................................+......................................................................................................................................+....................................................................................................................................................................................................+..............................................................................................................................................+.............................+............................................................................................................................................................+.......................................................................+.......................................................+..............................................................................+.......................................+.+............................................+..........+.................................+...........+...........................................................................................................................................................................................................................................................................................+.....+.......................................................................................................+........................................................+......................................................................+..........................................................................+...............................................+.....+.......................................................................................................................+...................................................................................+..............................................+.........................+..............................................................................................................+.................................................+.............................................................................................................+..+......................................................................................................................................+...................................................................................................................................................+......................................................................+..............................................................................................................................................................................................+.................................................................+.......................................................+............................................................................................................+...+....................................................................................................................................................................+......................................................................................................+...................................................+..................+......+....................................................................................................................................+..............................................+....................+.................+.........+.........................+..........................+........................+............................................................................................+..........................................................................................+.......+..........................................................................+.................................................+............................................................................................................................................................................................................................+................................+............................................................+..........................................................................................+........................................................................................................................+.....................................+........................................+........+......................................................................+.................................................................................................................................................................+.......................+......................................................................+.....................................................................................................................+.....++*++*++*++*

DH parameters of size 2048 created at /etc/openvpn/easy-rsa/pki/dh.pem
```

#### Generar certificado + clave privada del cliente VPN

```
sudo ./easyrsa build-client-full clientevpn nopass
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021
Generating a RSA private key
...........................................................................................................................................................................+++++
..................................................+++++
writing new private key to '/etc/openvpn/easy-rsa/pki/easy-rsa-22100.RLN0zO/tmp.EUi7fV'
-----
Using configuration from /etc/openvpn/easy-rsa/pki/easy-rsa-22100.RLN0zO/tmp.7aLPgE
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'clientevpn'
Certificate is to be certified until Mar 20 07:38:48 2024 GMT (825 days)

Write out database with 1 new entries
Data Base Updated
```

#### Pasar ficheros necesarios al cliente VPN

```
sudo cp -rp /etc/openvpn/easy-rsa/pki/{ca.crt,issued/clientevpn.crt,private/clientevpn.key} ~
sudo chown vagrant: ~/{ca.crt,clientevpn.crt,clientevpn.key}
scp ~/{ca.crt,clientevpn.crt,clientevpn.key} vagrant@192.88.99.11:/home/vagrant
```

#### Configurar el servidor VPN

Creo el `client-config-dir` junto con un fichero con información sobre `clientevpn`:
```
cd /etc/openvpn
sudo mkdir ccd
cd ccd
sudo touch clientevpn
echo iroute 192.168.0.0 255.255.255.0 > /etc/openvpn/ccd/clientevpn
```

Creo el fichero de configuración partiendo de una plantilla:
```
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/servidor.conf
```

Modifico `/etc/openvpn/server/servidor.conf`:
```
port 1194
proto udp
dev tun


ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem


topology subnet


server 10.99.99.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt


push "route 192.168.1.0 255.255.255.0"

client-config-dir /etc/openvpn/ccd
route 192.168.0.0 255.255.255.0


keepalive 10 120
cipher AES-256-CBC
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
```

#### Iniciar el servidor

```
sudo systemctl enable --now openvpn-server@servidor
```

Verifico que está funcionando:
```
sudo systemctl status openvpn-server@servidor
```
```
● openvpn-server@servidor.service - OpenVPN service for servidor
     Loaded: loaded (/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-16 11:09:16 UTC; 48s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 22955 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 528)
     Memory: 1.0M
        CPU: 12ms
     CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@servidor.service
             └─22955 /usr/sbin/openvpn --status /run/openvpn-server/status-servidor.log --status-version 2 --suppress-timestamps --config servidor.conf

Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 MULTI_sva: pool returned IPv4=10.99.99.2, IPv6=(Not enabled)
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 OPTIONS IMPORT: reading client specific options from: /etc/openvpn/ccd/clientevpn
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 MULTI: Learn: 10.99.99.2 -> clientevpn/192.88.99.11:55317
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 MULTI: primary virtual IP for clientevpn/192.88.99.11:55317: 10.99.99.2
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 MULTI: internal route 192.168.0.0/24 -> clientevpn/192.88.99.11:55317
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 MULTI: Learn: 192.168.0.0/24 -> clientevpn/192.88.99.11:55317
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 Data Channel: using negotiated cipher 'AES-256-GCM'
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Dec 16 11:09:19 servervpn openvpn[22955]: clientevpn/192.88.99.11:55317 SENT CONTROL [clientevpn]: 'PUSH_REPLY,route 192.168.1.0 255.255.255.0,route-gate>
```



### En clientevpn

#### Instalar OpenVPN

```
sudo apt update
sudo apt install openvpn
```

#### Habilitar forwarding

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### Mover ficheros a la ruta adecuada

```
sudo mv ~/{ca.crt,clientevpn.crt,clientevpn.key} /etc/openvpn/client
sudo chown root: /etc/openvpn/client/*
```

#### Configurar el cliente VPN

Copio la plantilla oficial de configuración para clientes openvpn, para partir de ella:
```
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client/cliente.conf
```

Modifico `/etc/openvpn/client/cliente.conf`:
```
client
dev tun
proto udp

remote 192.88.99.10 1194
resolv-retry infinite
nobind

persist-key
persist-tun

ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/clientevpn.crt
key /etc/openvpn/client/clientevpn.key

remote-cert-tls server
cipher AES-256-CBC
verb 3
```

Habilito el servicio:
```
sudo systemctl enable --now openvpn-client@cliente
```

Verifico que funciona:
```
sudo systemctl status openvpn-client@cliente
```
```
● openvpn-client@cliente.service - OpenVPN tunnel for cliente
     Loaded: loaded (/lib/systemd/system/openvpn-client@.service; enabled; vendor preset: enab>
     Active: active (running) since Thu 2021-12-16 09:07:50 UTC; 22s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 11823 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 528)
     Memory: 2.1M
        CPU: 35ms
     CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@cliente.service
             └─11823 /usr/sbin/openvpn --suppress-timestamps --nobind --config cliente.conf

Dec 16 09:07:50 clientevpn openvpn[11823]: net_route_v4_best_gw query: dst 0.0.0.0
Dec 16 09:07:50 clientevpn openvpn[11823]: net_route_v4_best_gw result: via 192.168.121.1 dev >
Dec 16 09:07:50 clientevpn openvpn[11823]: ROUTE_GATEWAY 192.168.121.1/255.255.255.0 IFACE=eth>
Dec 16 09:07:50 clientevpn openvpn[11823]: TUN/TAP device tun0 opened
Dec 16 09:07:50 clientevpn openvpn[11823]: net_iface_mtu_set: mtu 1500 for tun0
Dec 16 09:07:50 clientevpn openvpn[11823]: net_iface_up: set tun0 up
Dec 16 09:07:50 clientevpn openvpn[11823]: net_addr_v4_add: 10.99.99.2/24 dev tun0
Dec 16 09:07:50 clientevpn openvpn[11823]: net_route_v4_add: 192.168.1.0/24 via 10.99.99.1 dev>
Dec 16 09:07:50 clientevpn openvpn[11823]: WARNING: this configuration may cache passwords in >
Dec 16 09:07:50 clientevpn openvpn[11823]: Initialization Sequence Completed
```



### En clientenormal1

#### Cambiar ruta por defecto

```
sudo ip route del default
sudo ip route add default via 192.168.0.1
```



### En clientenormal2

#### Cambiar ruta por defecto

```
sudo ip route del default
sudo ip route add default via 192.168.1.1
```



### Pruebas de funcionamiento

Ping desde `clientenormal1` a `clientenormal2`:
```
vagrant@clientenormal1:~$ ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=62 time=1.03 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=62 time=1.25 ms
^C
--- 192.168.1.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 1.028/1.140/1.253/0.112 ms
```

Ping desde `clientenormal2` a `clientenormal1`:
```
vagrant@clientenormal2:~$ ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=2.39 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=62 time=0.958 ms
^C
--- 192.168.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.958/1.673/2.389/0.715 ms
```

Traceroute desde `clientenormal1` a `clientenormal2`:
```
vagrant@clientenormal1:~$ traceroute 192.168.1.2
traceroute to 192.168.1.2 (192.168.1.2), 30 hops max, 60 byte packets
 1  192.168.0.1 (192.168.0.1)  0.304 ms  0.265 ms  0.254 ms
 2  10.99.99.1 (10.99.99.1)  1.239 ms  1.218 ms  3.301 ms
 3  192.168.1.2 (192.168.1.2)  3.624 ms  3.692 ms  3.684 ms
```

Traceroute desde `clientenormal2` a `clientenormal1`:
```
vagrant@clientenormal2:~$ traceroute 192.168.0.2
traceroute to 192.168.0.2 (192.168.0.2), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  0.478 ms  0.350 ms  0.342 ms
 2  10.99.99.2 (10.99.99.2)  3.005 ms  2.959 ms  2.931 ms
 3  192.168.0.2 (192.168.0.2)  2.924 ms  2.898 ms  2.888 ms
```






## Parte 3
 VPN de acceso remoto con WireGuard (10 puntos)

Monta una VPN de acceso remoto usando Wireguard. Intenta probarla con clientes Windows, Linux y Android. Documenta el proceso adecuadamente y compáralo con el del apartado A.

(comparación de configuración, facilidad, básicamente un apartado final de conclusiones)



## Parte 4
 VPN sitio a sitio con WireGuard (10 puntos)

Configura una VPN sitio a sitio usando WireGuard. Documenta el proceso adecuadamente y compáralo con el del apartado B.







## Parte 5
 VPN de acceso remoto con Ipsec (5 puntos)

Elige una aplicación por software (por ejemplo, FreeS/Wan) y monta la configuración. Documenta el proceso detalladamente.

## Parte 6
 VPN sitio a sitio con IPsec (10 puntos)

Montando el escenario en GNS3 usando routers CISCO o con una aplicación por software (por ejemplo, FreeS/Wan) despliega la configuración solicitada. Documenta el proceso detalladamente.









































end
