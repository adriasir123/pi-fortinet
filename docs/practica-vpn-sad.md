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

![](https://i.postimg.cc/fynq296Q/site-to-site-openvpn.png)

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
..................+++++
..................................+++++
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
..............................................................+++++
........................................................................................................................+++++
writing new private key to '/etc/openvpn/easy-rsa/pki/easy-rsa-2046.nRVdyM/tmp.6vm2xf'
-----
Using configuration from /etc/openvpn/easy-rsa/pki/easy-rsa-2046.nRVdyM/tmp.3Nm9uS
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Apr  3 09:13:11 2024 GMT (825 days)

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
...................................+....................+..........................+....................................................................................................................................................................................................................+.....................+.....................................................................................................................................................................................................................................+...........+.................................................................................................................................+.................................................+....................................................+.........................................................................................................................................................................................................................+.............................+.............................................+..................................................................................................................................................................+............................................................................................................................................................................................................................................................................+..............................................................................................................................+.........................................................................................................+.......................................................................+................................................................................................................................................................+.......................................................................................................................................................+........................................................................................................................................................................................+...................................................................................+............................................................................+....................................+...............................................................+...............................................................................................................+.....................................................................................................................................................................................................................................+.....................+...................................+..................................................................................................+.............................................................................................+.......................................................................+..................................+......................................................................................................................................+....................................................+........................................................................................................................................................................................................................................................+........................................................................................................................................+...................................................................................................................................+....................................+....................+.....................................................................................................+..+...............................+..................................+...........................+.....................................................................................................................+...............................................................................................................................................................................................+................+...+.........................................................+..+.........+....+...+................................................................................................+...............................................................................................................................................................................................................................................................................................................................................+..................................................................+...............................................................................................................................+.....................................................................................................+............+...................................+.............................................................................................................................................+...............................................................................................+.........+.................+...........................................................................................................................................+....+........................................................................................+................+........+................................................................................................................................................................................................................................................+.......................................................................................................+.............................................................................................................................................................................................................................................+................................................................+...........................................................................................................................................................................+..................+..................................+...................................................................................................................................................................................................................................+....................................................................................................+.......................................................................................................................................................................................+..................................++*++*++*++*

DH parameters of size 2048 created at /etc/openvpn/easy-rsa/pki/dh.pem
```

#### Generar certificado + clave privada del cliente VPN

```
sudo ./easyrsa build-client-full clientevpn nopass
```
```
Using SSL: openssl OpenSSL 1.1.1k  25 Mar 2021
Generating a RSA private key
.......................................................................+++++
................................+++++
writing new private key to '/etc/openvpn/easy-rsa/pki/easy-rsa-2162.MUtxW0/tmp.cwHjMI'
-----
Using configuration from /etc/openvpn/easy-rsa/pki/easy-rsa-2162.MUtxW0/tmp.tJCZID
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'clientevpn'
Certificate is to be certified until Apr  3 09:28:02 2024 GMT (825 days)

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
     Active: active (running) since Thu 2021-12-30 09:47:53 UTC; 12s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 2291 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 528)
     Memory: 1020.0K
        CPU: 7ms
     CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@servidor.service
             └─2291 /usr/sbin/openvpn --status /run/openvpn-server/status-servidor.log --status-vers>

Dec 30 09:47:53 servervpn openvpn[2291]: net_addr_v4_add: 10.99.99.1/24 dev tun0
Dec 30 09:47:53 servervpn openvpn[2291]: net_route_v4_add: 192.168.0.0/24 via 10.99.99.2 dev [NULL] >
Dec 30 09:47:53 servervpn openvpn[2291]: Could not determine IPv4/IPv6 protocol. Using AF_INET
Dec 30 09:47:53 servervpn openvpn[2291]: Socket Buffers: R=[212992->212992] S=[212992->212992]
Dec 30 09:47:53 servervpn openvpn[2291]: UDPv4 link local (bound): [AF_INET][undef]:1194
Dec 30 09:47:53 servervpn openvpn[2291]: UDPv4 link remote: [AF_UNSPEC]
Dec 30 09:47:53 servervpn openvpn[2291]: MULTI: multi_init called, r=256 v=256
Dec 30 09:47:53 servervpn openvpn[2291]: IFCONFIG POOL IPv4: base=10.99.99.2 size=252
Dec 30 09:47:53 servervpn openvpn[2291]: IFCONFIG POOL LIST
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
     Loaded: loaded (/lib/systemd/system/openvpn-client@.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-30 10:20:45 UTC; 39s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 10909 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 528)
     Memory: 1.4M
        CPU: 11ms
     CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@cliente.service
             └─10909 /usr/sbin/openvpn --suppress-timestamps --nobind --config cliente.conf

Dec 30 10:20:45 clientevpn openvpn[10909]: net_route_v4_best_gw query: dst 0.0.0.0
Dec 30 10:20:45 clientevpn openvpn[10909]: net_route_v4_best_gw result: via 192.168.121.1 dev eth0
Dec 30 10:20:45 clientevpn openvpn[10909]: ROUTE_GATEWAY 192.168.121.1/255.255.255.0 IFACE=eth0 HWADDR=52:54:00:>
Dec 30 10:20:45 clientevpn openvpn[10909]: TUN/TAP device tun0 opened
Dec 30 10:20:45 clientevpn openvpn[10909]: net_iface_mtu_set: mtu 1500 for tun0
Dec 30 10:20:45 clientevpn openvpn[10909]: net_iface_up: set tun0 up
Dec 30 10:20:45 clientevpn openvpn[10909]: net_addr_v4_add: 10.99.99.2/24 dev tun0
Dec 30 10:20:45 clientevpn openvpn[10909]: net_route_v4_add: 192.168.1.0/24 via 10.99.99.1 dev [NULL] table 0 me>
Dec 30 10:20:45 clientevpn openvpn[10909]: WARNING: this configuration may cache passwords in memory -- use the >
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
64 bytes from 192.168.1.2: icmp_seq=1 ttl=62 time=4.45 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=62 time=2.30 ms
^C
--- 192.168.1.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 2.301/3.376/4.451/1.075 ms
```

Ping desde `clientenormal2` a `clientenormal1`:
```
vagrant@clientenormal2:~$ ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=2.26 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=62 time=2.24 ms
64 bytes from 192.168.0.2: icmp_seq=3 ttl=62 time=2.19 ms
^C
--- 192.168.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 2.188/2.229/2.264/0.031 ms
```

Traceroute desde `clientenormal1` a `clientenormal2`:
```
vagrant@clientenormal1:~$ traceroute 192.168.1.2
traceroute to 192.168.1.2 (192.168.1.2), 30 hops max, 60 byte packets
 1  192.168.0.1 (192.168.0.1)  0.521 ms  0.438 ms  0.433 ms
 2  10.99.99.1 (10.99.99.1)  2.229 ms  2.196 ms  2.177 ms
 3  192.168.1.2 (192.168.1.2)  2.129 ms  2.082 ms  2.035 ms
```

Traceroute desde `clientenormal2` a `clientenormal1`:
```
vagrant@clientenormal2:~$ traceroute 192.168.0.2
traceroute to 192.168.0.2 (192.168.0.2), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  0.613 ms  0.538 ms  0.507 ms
 2  10.99.99.2 (10.99.99.2)  2.864 ms  2.828 ms  2.812 ms
 3  192.168.0.2 (192.168.0.2)  2.824 ms  2.778 ms  3.276 ms
```






## Parte 3: acceso remoto WireGuard

### Escenario

![](https://i.postimg.cc/xCnCxbR6/acceso-remoto-wireguard.jpg)

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
      :libvirt__network_name => "internet-vpn3",
      :libvirt__dhcp_enabled => false,
      :ip => "192.88.99.11",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servervpn do |servervpn|
    servervpn.vm.box = "debian/bullseye64"
    servervpn.vm.hostname = "servervpn"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "internet-vpn3",
      :libvirt__dhcp_enabled => false,
      :ip => "192.88.99.10",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "privada-vpn3",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientenormal do |clientenormal|
    clientenormal.vm.box = "debian/bullseye64"
    clientenormal.vm.hostname = "clientenormal"
    clientenormal.vm.network :private_network,
      :libvirt__network_name => "privada-vpn3",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```



### En servervpn

#### Instalar WireGuard

```
sudo apt update
sudo apt install wireguard
```

#### Generar par de claves

```
wg genkey | sudo tee /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key
```

#### Configurar

Creo el fichero de configuración *(wg0 será el nombre de la interfaz)*:
```
sudo touch /etc/wireguard/wg0.conf
```

Lo modifico:
```
[Interface]
Address = 10.99.99.1/24
ListenPort = 51820
PrivateKey = wPO8E5rUB9hFCwJMQYsLwDtNV7eX9SZmL9h/VO+NwFY=

# IP forwarding
PreUp = sysctl -w net.ipv4.ip_forward=1

[Peer]
PublicKey = GLJ8rEoPu22e+t2xPP54TJoGCoq09+HiOuiwBV3TzSw=
AllowedIPs = 10.99.99.2/32
```

#### Modificar permisos

```
sudo chmod 600 /etc/wireguard/ -R
```

#### Iniciar WireGuard

```
sudo wg-quick up /etc/wireguard/wg0.conf
```
```
[#] sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.99.99.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
```

Se nos crea la interfaz:
```
5: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.99.99.1/24 scope global wg0
       valid_lft forever preferred_lft forever
```

Y la ruta:
```
10.99.99.0/24 dev wg0 proto kernel scope link src 10.99.99.1
```



### En clientevpn (debian 11)

#### Instalar WireGuard

```
sudo apt update
sudo apt install wireguard
```

#### Generar par de claves

```
wg genkey | sudo tee /etc/wireguard/client_private.key | wg pubkey | sudo tee /etc/wireguard/client_public.key
```

#### Configurar

Creo el fichero de configuración *(wg-client0 será el nombre de la interfaz)*:
```
sudo touch /etc/wireguard/wg-client0.conf
```

Lo modifico:
```
[Interface]
Address = 10.99.99.2/24
PrivateKey = iO5lDgbhB6cGsJL6Udw0SP+9x4N0FHrRj2irMiNM93Y=

[Peer]
PublicKey = K7noXpmRF60rW7BjqGTHTF8PaOVLASb1UJJF62YsFw8=
AllowedIPs = 0.0.0.0/0
Endpoint = 192.88.99.10:51820
PersistentKeepalive = 25
```

#### Modificar permisos

```
sudo chmod 600 /etc/wireguard/ -R
```

#### Iniciar WireGuard

```
sudo wg-quick up /etc/wireguard/wg-client0.conf
```
```
[#] ip link add wg-client0 type wireguard
[#] wg setconf wg-client0 /dev/fd/63
[#] ip -4 address add 10.99.99.2/24 dev wg-client0
[#] ip link set mtu 1420 up dev wg-client0
[#] wg set wg-client0 fwmark 51820
[#] ip -4 route add 0.0.0.0/0 dev wg-client0 table 51820
[#] ip -4 rule add not fwmark 51820 table 51820
[#] ip -4 rule add table main suppress_prefixlength 0
[#] sysctl -q net.ipv4.conf.all.src_valid_mark=1
[#] nft -f /dev/fd/63
```

Se nos crea la interfaz:
```
4: wg-client0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.99.99.2/24 scope global wg-client0
       valid_lft forever preferred_lft forever
```

Y la ruta:
```
10.99.99.0/24 dev wg-client0 proto kernel scope link src 10.99.99.2
```



### En clientenormal

#### Cambiar ruta por defecto

```
sudo ip route del default
sudo ip route add default via 192.168.0.1
```



### Pruebas de funcionamiento

*(todas hechas desde clientevpn)*

Ping al otro extremo del túnel:
```
vagrant@clientevpn:~$ ping 10.99.99.1
PING 10.99.99.1 (10.99.99.1) 56(84) bytes of data.
64 bytes from 10.99.99.1: icmp_seq=1 ttl=64 time=2.43 ms
64 bytes from 10.99.99.1: icmp_seq=2 ttl=64 time=0.232 ms
^C
--- 10.99.99.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.232/1.328/2.425/1.096 ms
```

Ping a clientenormal:
```
vagrant@clientevpn:~$ ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=63 time=1.56 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=63 time=0.360 ms
^C
--- 192.168.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.360/0.959/1.559/0.599 ms
```

Traceroute a clientenormal:
```
vagrant@clientevpn:~$ traceroute 192.168.0.2
traceroute to 192.168.0.2 (192.168.0.2), 30 hops max, 60 byte packets
 1  10.99.99.1 (10.99.99.1)  0.734 ms  0.275 ms  0.242 ms
 2  192.168.0.2 (192.168.0.2)  0.712 ms  0.664 ms  0.617 ms
```



### Extra: cliente VPN Windows

#### Conectar cliente

Conecto la VM a la red `internet-vpn3`:

![](https://i.imgur.com/oeLET6E.png)

Configuro estáticamente la interfaz:

```
netsh interface ip set address name="Ethernet" static 192.88.99.12 255.255.255.0 192.88.99.10
```

Compruebo que la configuración se aplicó correctamente:

![](https://i.imgur.com/RmHnYNI.png)

Compruebo que tengo conectividad con servervpn:

![](https://i.imgur.com/kC4Rl5T.png)

#### Configurar cliente

Descargo e instalo Wireguard:

![](https://i.imgur.com/PnuNBVQ.png)

Compruebo que funciona el programa:

![](https://i.imgur.com/6xiDTEw.png)

Creo un nuevo túnel en blanco:

![](https://i.imgur.com/MQs39Ts.png)

Como nos podemos dar cuenta, la generación de claves ha sido automática:

![](https://i.imgur.com/zMqkR69.png)

Completo la configuración y nombro el túnel:

![](https://i.imgur.com/NuWRSBw.png)

Verifico que se ha creado correctamente:

![](https://i.imgur.com/YRScpHp.png)

Nos damos cuenta de que está inactivo (por ahora).

#### Modificar el servidor

Tengo que añadir el nuevo peer a `wg0.conf`. El fichero se quedaría así:
```
[Interface]
Address = 10.99.99.1/24
ListenPort = 51820
PrivateKey = wPO8E5rUB9hFCwJMQYsLwDtNV7eX9SZmL9h/VO+NwFY=

# IP forwarding
PreUp = sysctl -w net.ipv4.ip_forward=1

[Peer]
PublicKey = GLJ8rEoPu22e+t2xPP54TJoGCoq09+HiOuiwBV3TzSw=
AllowedIPs = 10.99.99.2/32

[Peer]
PublicKey = 3CNoT/Zae2eXK+yW0wm13qFWOEOwUcfCQrcIGttUPyk=
AllowedIPs = 10.99.99.3/32
```

Reinicio el servidor para que los cambios tomen efecto:
```
sudo wg-quick down /etc/wireguard/wg0.conf
sudo wg-quick up /etc/wireguard/wg0.conf
```

#### Activar cliente

![](https://i.imgur.com/muCEm1S.png)

![](https://i.imgur.com/68Ar9SV.png)

Vemos que se nos crea la interfaz de túnel:

![](https://i.imgur.com/42ytjaP.png)

#### Pruebas de funcionamiento

Ping al otro extremo del túnel:

![](https://i.imgur.com/XR4fuGJ.png)

Ping a clientenormal:

![](https://i.imgur.com/vikD4lO.png)

Tracert a clientenormal:

![](https://i.imgur.com/pwMVQ8J.png)



### Extra: cliente VPN Android

#### Conectar cliente

Conecto la VM a la red `internet-vpn3`:

![](https://i.imgur.com/GZCPo8P.png)

Configuro estáticamente la interfaz:

```
ifconfig wlan0 192.88.99.13 netmask 255.255.255.0 up
```

Compruebo que la configuración se aplicó correctamente:

![](https://i.imgur.com/UpCkiZu.png)

Compruebo que tengo conectividad con servervpn:

![](https://i.imgur.com/KE9LR9G.png)

#### Configurar cliente

Instalo Wireguard:

![](https://i.imgur.com/1vEHzQN.png)

Compruebo que funciona el programa:

![](https://i.imgur.com/t2LZRNB.png)

Creo un nuevo túnel en blanco:

![](https://i.imgur.com/iS2mELB.png)

Genero un par de claves:

![](https://i.imgur.com/9PAPaVg.png)

Completo la configuración:

![](https://i.imgur.com/fITSxMx.png)

![](https://i.imgur.com/G6KNIlD.png)

Verifico que se ha creado correctamente el túnel:

![](https://i.imgur.com/vqUOuEw.png)

Está inactivo (por ahora).

#### Modificar el servidor

Tengo que añadir el nuevo peer a `wg0.conf`. El fichero se quedaría así:
```
[Interface]
Address = 10.99.99.1/24
ListenPort = 51820
PrivateKey = wPO8E5rUB9hFCwJMQYsLwDtNV7eX9SZmL9h/VO+NwFY=

# IP forwarding
PreUp = sysctl -w net.ipv4.ip_forward=1

[Peer]
PublicKey = GLJ8rEoPu22e+t2xPP54TJoGCoq09+HiOuiwBV3TzSw=
AllowedIPs = 10.99.99.2/32

[Peer]
PublicKey = 3CNoT/Zae2eXK+yW0wm13qFWOEOwUcfCQrcIGttUPyk=
AllowedIPs = 10.99.99.3/32

[Peer]
PublicKey = DJusByULiqhbzBhSCDdVJZAjH6mNEUVE9qlyYc/CylM=
AllowedIPs = 10.99.99.4/32
```

Reinicio el servidor para que los cambios tomen efecto:
```
sudo wg-quick down /etc/wireguard/wg0.conf
sudo wg-quick up /etc/wireguard/wg0.conf
```

#### Activar cliente

![](https://i.imgur.com/Mkp7WxP.png)

![](https://i.imgur.com/1amrzI9.png)

Vemos que se nos crea la interfaz de túnel:

![](https://i.imgur.com/kFAx18Z.png)

#### Pruebas de funcionamiento

Ping al otro extremo del túnel:

![](https://i.imgur.com/1acerxo.png)

Ping a clientenormal:

![](https://i.imgur.com/3X9kygw.png)

Traceroute a clientenormal:

![](https://i.imgur.com/y5hyVPP.png)



### Conclusiones

Aquí expondré las conclusiones a las que he llegado tras comparar el acceso remoto de WireGuard con el de OpenVPN.

* Con WireGuard no existe realmente el concepto de cliente-servidor, sino el de peer-to-peer. Aunque en la práctica, se distingue fácilmente cuál es el "servidor" porque es en el que abrimos un puerto específico durante la configuración.

* La autenticación es mucho más sencilla con WireGuard, ya que se usa un par de claves y no certificados. Conceptualmente ambos mecanismos son casi iguales, pero en la práctica es mucho más cómodo trabajar con un par de claves porque nos ahorramos el tener que crear una Autoridad Certificadora, el firmado de certificados...etc. Es muy parecido, por no decir casi igual, a como se trabaja con SSH.

* Los ficheros de configuración son notoriamente más cortos y sencillos en WireGuard.

* WireGuard nos proporciona la herramienta `wg-quick` para controlar el servicio, lo cual es mucho más sencillo que trabajar con systemd.






## Parte 4: sitio a sitio WireGuard

### Escenario

![](https://i.postimg.cc/4yKyNf9V/site-to-site-wireguard.jpg)

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
      :libvirt__network_name => "privada1-vpn4",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :peervpn1 do |peervpn1|
    peervpn1.vm.box = "debian/bullseye64"
    peervpn1.vm.hostname = "peervpn1"
    peervpn1.vm.network :private_network,
      :libvirt__network_name => "privada1-vpn4",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    peervpn1.vm.network :private_network,
      :libvirt__network_name => "internet-vpn4",
      :libvirt__dhcp_enabled => false,
      :ip => "192.88.99.11",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :peervpn2 do |peervpn2|
    peervpn2.vm.box = "debian/bullseye64"
    peervpn2.vm.hostname = "peervpn2"
    peervpn2.vm.network :private_network,
      :libvirt__network_name => "internet-vpn4",
      :libvirt__dhcp_enabled => false,
      :ip => "192.88.99.10",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    peervpn2.vm.network :private_network,
      :libvirt__network_name => "privada2-vpn4",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.1.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientenormal2 do |clientenormal2|
    clientenormal2.vm.box = "debian/bullseye64"
    clientenormal2.vm.hostname = "clientenormal2"
    clientenormal2.vm.network :private_network,
      :libvirt__network_name => "privada2-vpn4",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.1.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```



### En peervpn1

#### Instalar WireGuard

```
sudo apt update
sudo apt install wireguard
```

#### Generar par de claves

```
wg genkey | sudo tee /etc/wireguard/peervpn1_private.key | wg pubkey | sudo tee /etc/wireguard/peervpn1_public.key
```

#### Configurar

Creo el fichero de configuración:
```
sudo touch /etc/wireguard/wg0.conf
```

Lo modifico:
```
[Interface]
Address = 10.99.99.1/32
ListenPort = 51820
PrivateKey = 4FMOxWlQLSGHlfgjFKBkmPDeURsHh7RBeCAoYMd/vGw=

# IP forwarding
PreUp = sysctl -w net.ipv4.ip_forward=1

[Peer]
Endpoint = 192.88.99.10:51820
AllowedIPs = 10.99.99.2/32, 192.168.1.0/24
PublicKey = FMJMI1Cg7jOLPTZj49DP/KE7RVK9/J8X5l3KG4LVf2Q=
```

#### Modificar permisos

```
sudo chmod 600 /etc/wireguard/ -R
```

#### Iniciar WireGuard

```
sudo wg-quick up /etc/wireguard/wg0.conf
```
```
[#] sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.99.99.1/32 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] ip -4 route add 10.99.99.2/32 dev wg0
[#] ip -4 route add 192.168.1.0/24 dev wg0
```

Se crea la interfaz:
```
7: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.99.99.1/32 scope global wg0
       valid_lft forever preferred_lft forever
```

Y las rutas:
```
10.99.99.2 dev wg0 scope link
192.168.1.0/24 dev wg0 scope link
```



### En peervpn2

#### Instalar WireGuard

```
sudo apt update
sudo apt install wireguard
```

#### Generar par de claves

```
wg genkey | sudo tee /etc/wireguard/peervpn2_private.key | wg pubkey | sudo tee /etc/wireguard/peervpn2_public.key
```

#### Configurar

Creo el fichero de configuración:
```
sudo touch /etc/wireguard/wg0.conf
```

Lo modifico:
```
[Interface]
Address = 10.99.99.2/32
ListenPort = 51820
PrivateKey = UP7xLuRK2JtsFxUwWqHxeREAwKPQPO9mvYfI+neNd3E=

# IP forwarding
PreUp = sysctl -w net.ipv4.ip_forward=1

[Peer]
Endpoint = 192.88.99.11:51820
AllowedIPs = 10.99.99.1/32, 192.168.0.0/24
PublicKey = nN/tik1TNOkFuupFVlNOaEO5hwQFKlsUWoKYsSCutzA=
```

#### Modificar permisos

```
sudo chmod 600 /etc/wireguard/ -R
```

#### Iniciar WireGuard

```
sudo wg-quick up /etc/wireguard/wg0.conf
```
```
[#] sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.99.99.2/32 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] ip -4 route add 10.99.99.1/32 dev wg0
[#] ip -4 route add 192.168.0.0/24 dev wg0
```

Se nos crea la interfaz:
```
7: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.99.99.2/32 scope global wg0
       valid_lft forever preferred_lft forever
```

Y las rutas:
```
10.99.99.1 dev wg0 scope link
192.168.0.0/24 dev wg0 scope link
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
64 bytes from 192.168.1.2: icmp_seq=1 ttl=62 time=2.77 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=62 time=0.958 ms
^C
--- 192.168.1.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.958/1.865/2.772/0.907 ms
```

Ping desde `clientenormal2` a `clientenormal1`:
```
vagrant@clientenormal2:~$ ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=1.26 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=62 time=0.721 ms
^C
--- 192.168.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.721/0.992/1.264/0.271 ms
```

Traceroute desde `clientenormal1` a `clientenormal2`:
```
vagrant@clientenormal1:~$ traceroute 192.168.1.2
traceroute to 192.168.1.2 (192.168.1.2), 30 hops max, 60 byte packets
 1  192.168.0.1 (192.168.0.1)  0.264 ms  0.244 ms  0.228 ms
 2  10.99.99.2 (10.99.99.2)  0.558 ms  0.540 ms  0.495 ms
 3  192.168.1.2 (192.168.1.2)  0.748 ms  0.700 ms  0.685 ms
```

Traceroute desde `clientenormal2` a `clientenormal1`:
```
vagrant@clientenormal2:~$ traceroute 192.168.0.2
traceroute to 192.168.0.2 (192.168.0.2), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  0.800 ms  0.774 ms  0.764 ms
 2  10.99.99.1 (10.99.99.1)  1.665 ms  1.653 ms  1.760 ms
 3  192.168.0.2 (192.168.0.2)  2.553 ms  2.544 ms  2.535 ms
```



### Conclusiones

Aquí expondré las conclusiones a las que he llegado tras comparar el sitio a sitio de WireGuard con el de OpenVPN.  

Además de las ya mencionadas en el anterior apartado de conclusiones, sólo quedaría una más por destacar, y que sólo ha sucedido con la configuración de sitio a sitio:

* El manejo de rutas es mucho más sencillo con WireGuard, ya que sólo nos tenemos que preocupar de que el parámetro `AllowedIPs` esté correctamente relleno con las rutas que necesitamos en cada peer. Con OpenVPN influyen los `ccd`, "complicando" un poco la configuración.






## Bibliografía

PARTE 1:

<https://kifarunix.com/install-openvpn-server-on-debian-11-debian-10/>

PARTE 2:

<https://medium.com/@bjammal/site-to-site-vpn-on-a-single-host-using-openvpn-e9c5cdb22f92>
<https://www.matoski.com/article/site-to-site-openvpn-firewalld-debian/>  
<https://openvpn.net/community-resources/how-to/#scope> (sección *Including multiple machines on the client side when using a routed VPN (dev tun)*)

PARTE 3:

<https://www.linuxbabe.com/debian/wireguard-vpn-server-debian>  
<https://www.makeuseof.com/how-to-set-up-wireguard-windows/>

PARTE 4:

<https://www.procustodibus.com/blog/2020/12/wireguard-site-to-site-config/>
