# Ejercicio 2: Reserva DHCP

## ENTREGA

### Parte 1
> Crear una reserva en 192.168.0.105 para cliente2

Añadir a `/etc/dhcp/dhcpd.conf` el siguiente bloque:
```
host cliente2 {
  hardware ethernet 52:54:00:78:7e:d7;
  fixed-address 192.168.0.105;
}
```

Hay 2 únicos campos importantes:

- `hardware ethernet`: MAC de la interfaz en cliente2 a recibir la IP fija
- `fixed-address`: IP fija que recibirá cliente2

Reiniciamos el servicio
```
sudo systemctl restart isc-dhcp-server.service
```

### Parte 2
> Hacer `ip a` en cliente2

```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:78:7e:d7 brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet 192.168.0.105/24 brd 192.168.0.255 scope global dynamic eth1
       valid_lft 3568sec preferred_lft 3568sec
    inet6 fe80::5054:ff:fe78:7ed7/64 scope link
       valid_lft forever preferred_lft forever
```

### Parte 3
> ¿Se guarda una concesión en `/var/lib/dhcp/dhcpd.leases` para la reserva hecha?

Como mostraré a continuación en mi fichero: *no, no se guarda*.
```
cat /var/lib/dhcp/dhcpd.leases
```
```
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

lease 192.168.0.100 {
  starts 1 2021/10/18 16:15:09;
  ends 1 2021/10/18 16:28:50;
  tstp 1 2021/10/18 16:28:50;
  cltt 1 2021/10/18 16:15:09;
  binding state free;
  hardware ethernet 52:54:00:ac:7b:64;
  uid "\377\000\254{d\000\001\000\001)\000I\365RT\000\254{d";
}
lease 192.168.0.102 {
  starts 1 2021/10/18 21:54:59;
  ends 1 2021/10/18 22:54:59;
  tstp 1 2021/10/18 22:54:59;
  cltt 1 2021/10/18 21:54:59;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 52:54:00:78:7e:d7;
  uid "\377\000x~\327\000\001\000\001)\000\251\262RT\000x~\327";
  client-hostname "cliente2";
}
lease 192.168.0.101 {
  starts 1 2021/10/18 22:00:32;
  ends 1 2021/10/18 23:00:32;
  tstp 1 2021/10/18 23:00:32;
  cltt 1 2021/10/18 22:00:32;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 52:54:00:ac:7b:64;
  client-hostname "cliente";
}
server-duid "\000\001\000\001)\000\255.RT\000V\3362";

lease 192.168.0.102 {
  starts 1 2021/10/18 21:54:59;
  ends 1 2021/10/18 22:24:37;
  tstp 1 2021/10/18 22:24:37;
  cltt 1 2021/10/18 21:54:59;
  binding state free;
  hardware ethernet 52:54:00:78:7e:d7;
  uid "\377\000x~\327\000\001\000\001)\000\251\262RT\000x~\327";
}
lease 192.168.0.101 {
  starts 1 2021/10/18 22:27:03;
  ends 1 2021/10/18 23:27:03;
  cltt 1 2021/10/18 22:27:03;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 52:54:00:ac:7b:64;
  client-hostname "cliente";
}
```

La razón es que este fichero **sólo almacena las "*dynamic leases*", no las reservas**.


## Escenario
Cambios en Vagrantfile con respecto al escenario anterior en "ejercicio 1 DHCP":
```
config.vm.define :cliente2 do |cliente2|
  cliente2.vm.box = "debian/bullseye64"
  cliente2.vm.hostname = "cliente2"
  cliente2.vm.network :private_network,
    :libvirt__network_name => "ej1-dhcp",
    :libvirt__dhcp_enabled => false,
    :libvirt__forward_mode => "veryisolated"
end
```


## Realización
> Pedir nueva configuración DHCP en cliente2

```
sudo dhclient -r eth1
sudo dhclient eth1
```
