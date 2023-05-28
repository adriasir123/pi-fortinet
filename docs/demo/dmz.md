# DMZ

## Paso 1

Cambiamos la interfaz de la Raspberry a su puerto definitivo, el de DMZ:

![61](../images/demo/61.jpeg)

Comprobamos desde la interfaz web que la DMZ está activada:

![62](../images/demo/62.png)

**La IP que vemos NO ES DE LA RASPBERRY**, es de la interfaz DMZ del FortiGate.

## Paso 2

Si vamos a la Raspberry y miramos las interfaces, nos vamos a dar cuenta de que nos ha dado APIPA:

![63](../images/demo/63.jpeg)

Esto es CORRECTO. Este funcionamiento es intencional, porque generalmente no querremos tener IPs dinámicas en una zona DMZ que estará siendo accesible públicamente continuamente.

De hecho, si nos fijamos en las posibles configuraciones de la interfaz DMZ, el FortiGate ni siquiera nos deja habilitar un servidor DHCP:

![64](../images/demo/64.png)

¿Cómo lo solucionamos? Tendremos que asignar la IP de forma estática manualmente dentro del rango.

Dejamos `/etc/network/interfaces` de la siguiente manera:

```shell
sudo nano /etc/network/interfaces
```

```shell
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback
 
# The primary network interface
auto eth0
iface eth0 inet static
 address 10.10.10.2
 netmask 255.255.255.0
 gateway 10.10.10.1
 dns-nameservers 8.8.8.8 8.8.4.4
```

También modificamos el siguiente fragmento en `/etc/dhcpcd.conf`:

```shell
sudo nano /etc/dhcpcd.conf
```

```shell
# Example static IP configuration:

interface eth0
static ip_address=10.10.10.2/24
static routers=10.10.10.1
static domain_name_servers= 8.8.8.8 8.8.4.4
static domain_search=
```

Reiniciamos networking y la interfaz:

```shell
sudo systemctl restart networking
sudo ifdown eth0
sudo ifup eth0
```

Comprobamos que tenemos la IP correcta y la puerta de enlace:

![65](../images/demo/65.jpeg)

Comprobamos que tenemos conectividad con la IP del FortiGate:

![66](../images/demo/66.jpeg)

## Paso 3














