# Taller 1: Configuración del cliente VPN

## Entregas

### Parte 1

IP de `tun0`:

![tun0](https://i.imgur.com/oHyX8PO.png)

Ruta a 172.22.0.0/16:

![ruta](https://i.imgur.com/hDE8reR.png)

Ping a la puerta de enlace:

![ping](https://i.imgur.com/n2mk6Bg.png)

### Parte 2

`/etc/hosts`:

![hosts](https://i.imgur.com/ijOmqEa.png)

Acceso a `openstack.gonzalonazareno.org` desde casa:

![openstackcasa](https://i.postimg.cc/cCdBQBCB/openstackcasa.gif)

## Realización

### Generación de clave privada

```shell
sudo sh -c 'openssl genrsa 4096 > /etc/ssl/private/olympus.key'
```

Modifico permisos:

```shell
sudo chmod 600 /etc/ssl/private/olympus.key
```

### Creación de csr

```shell
sudo openssl req -new -key /etc/ssl/private/olympus.key -out /root/olympus.csr
```

Introduzco los siguientes valores exactos en el certificado:

![datoscsr](https://i.imgur.com/eeIowf8.png)

Lo copio en mi home:

```shell
sudo cp /root/olympus.csr ~
```

### Paso de csr

Lo subo aquí: <https://dit.gonzalonazareno.org/gestiona/cert/>

![subidacsr](https://i.imgur.com/VuObEx7.png)

### Descarga de mi crt

![descargacrt](https://i.imgur.com/Tk2xNky.png)

Lo muevo al directorio correcto:

```shell
sudo mv ~/Downloads/olympus.crt /etc/openvpn
```

### Descarga del crt de la CA

Lo descargo aquí: <https://dit.gonzalonazareno.org/gestiona/info/documentacion/ca>

Lo muevo al directorio correcto:

```shell
sudo mv ~/Downloads/gonzalonazareno.crt /etc/ssl/certs
```

### Instalación de OpenVPN

```shell
sudo apt install openvpn
```

### Configuración de OpenVPN

Creo el siguiente fichero:

```shell
sudo nano /etc/openvpn/client.conf 
```

Con contenido:

```shell
dev tun
remote sputnik.gonzalonazareno.org
ifconfig 172.23.0.0 255.255.255.0
pull
proto tcp-client
tls-client
# remote-cert-tls server
ca /etc/ssl/certs/gonzalonazareno.crt
cert /etc/openvpn/olympus.crt
key /etc/ssl/private/olympus.key
comp-lzo
keepalive 10 60
log /var/log/openvpn-sputnik.log
verb 1
```

### Reinicio de OpenVPN

```shell
sudo systemctl restart openvpn
```

### Comprobaciones

Túnel creado:

![tun0](https://i.imgur.com/oHyX8PO.png)

Ruta añadida:

![ruta](https://i.imgur.com/hDE8reR.png)

Log:

```shell
 atlas@olympus  ~  sudo cat /var/log/openvpn-sputnik.log
2022-11-17 16:35:55 --cipher is not set. Previous OpenVPN version defaulted to BF-CBC as fallback when cipher negotiation failed in this case. If you need this fallback please add '--data-ciphers-fallback BF-CBC' to your configuration and/or add BF-CBC to --data-ciphers.
2022-11-17 16:35:55 OpenVPN 2.5.1 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on May 14 2021
2022-11-17 16:35:55 library versions: OpenSSL 1.1.1n  15 Mar 2022, LZO 2.10
2022-11-17 16:35:55 WARNING: using --pull/--client and --ifconfig together is probably not what you want
2022-11-17 16:35:55 WARNING: No server certificate verification method has been enabled.  See http://openvpn.net/howto.html#mitm for more info.
2022-11-17 16:35:55 TCP/UDP: Preserving recently used remote address: [AF_INET]5.39.73.79:1194
2022-11-17 16:35:55 Attempting to establish TCP connection with [AF_INET]5.39.73.79:1194 [nonblock]
2022-11-17 16:35:55 TCP connection established with [AF_INET]5.39.73.79:1194
2022-11-17 16:35:55 TCP_CLIENT link local: (not bound)
2022-11-17 16:35:55 TCP_CLIENT link remote: [AF_INET]5.39.73.79:1194
2022-11-17 16:35:55 [sputnik.gonzalonazareno.org] Peer Connection Initiated with [AF_INET]5.39.73.79:1194
2022-11-17 16:35:56 TUN/TAP device tun0 opened
2022-11-17 16:35:56 net_iface_mtu_set: mtu 1500 for tun0
2022-11-17 16:35:56 net_iface_up: set tun0 up
2022-11-17 16:35:56 net_addr_ptp_v4_add: 172.29.0.50 peer 172.29.0.49 dev tun0
2022-11-17 16:35:56 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
2022-11-17 16:35:56 Initialization Sequence Completed
```

### Ajustes finales

Deshabilito el autoarranque de OpenVPN:

```shell
sudo systemctl disable openvpn
```

Cada vez que lo queramos arrancar:

```shell
sudo systemctl start openvpn
```

Añado las siguientes entradas en`/etc/hosts`:

```shell
sudo nano /etc/hosts
```

```shell
172.22.123.100 openstack.gonzalonazareno.org
172.22.123.1 proxmox.gonzalonazareno.org
```
