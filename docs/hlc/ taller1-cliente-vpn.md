# Taller 1: Configuración del cliente VPN

## Generación de clave privada

```shell
sudo sh -c 'openssl genrsa 4096 > /etc/ssl/private/olympus.key'
```

## Creación de csr

```shell
sudo openssl req -new -key /etc/ssl/private/olympus.key -out /root/olympus.csr
```

Introduzco los siguientes valores exactos en el certificado:

![datoscsr](https://i.imgur.com/eeIowf8.png)

Lo copio en mi home:

```shell
sudo cp /root/olympus.csr ~
```

## Paso de csr

Lo subo aquí: <https://dit.gonzalonazareno.org/gestiona/cert/>

![subidacsr](https://i.imgur.com/VuObEx7.png)

## Una vez firmado el fichero csr aparecerá un fichero con extensión crt que se corresponde con el certificado firmado por la autoridad certificadora del Gonzalo Nazareno

Descargar el crt

![descargacrt](https://i.imgur.com/Tk2xNky.png)

Muevo al directorio correto:

```shell
sudo mv ~/Downloads/olympus.crt /etc/openvpn
```

## Descargar del crt del gonzalo

<https://dit.gonzalonazareno.org/gestiona/info/documentacion/ca>

Muevo al directorio correto:

```shell
sudo mv ~/Downloads/gonzalonazareno.crt /etc/ssl/certs
```

## Configuración de OpenVPN

## Instalamos el cliente OpenVPN

```shell
sudo apt install openvpn
```

## Creamos el siguiente fichero /etc/openvpn/client.conf con contenido

```shell
sudo nano /etc/openvpn/client.conf 
```

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

## Reinicia OpenVPN, comprueba que se haya creado el túnel y que se ha añadido una regla de encaminamiento adicional para acceder a los equipos de la 172.22.0.0/16: 

```shell
sudo systemctl restart openvpn
```

























