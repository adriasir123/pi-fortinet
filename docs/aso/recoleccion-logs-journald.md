# Recolección de logs centralizada mediante journald

## 1. Enunciado

Implementar un sistema de recolección de logs remoto mediante `systemd-journal-remote`.

## 2. Escenario

```ruby
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 2
    libvirt.memory = 1024
  end

  config.vm.define :server do |server|
    server.vm.box = "debian/bullseye64"
    server.vm.hostname = "server.adrianj.gonzalonazareno.org"
    server.vm.network :private_network,
      :libvirt__network_name => "recoleccion-logs",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :client do |client|
    client.vm.box = "debian/bullseye64"
    client.vm.hostname = "client.adrianj.gonzalonazareno.org"
    client.vm.network :private_network,
      :libvirt__network_name => "recoleccion-logs",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

## 3. Server

### 3.1 Paquetería

```shell
sudo apt update
sudo apt install systemd-journal-remote
```

### 3.2 Unidades systemd

```shell
sudo systemctl enable --now systemd-journal-remote.socket
sudo systemctl enable systemd-journal-remote.service
```

### 3.3 Configuración unidad

```shell
sudo systemctl edit systemd-journal-remote.service
```

```shell
[Service]
ExecStart=
ExecStart=/usr/lib/systemd/systemd-journal-remote --listen-http=-3 --output=/var/log/journal/remote/
```

```shell
sudo systemctl daemon-reload
```

### 3.4 Configuración servicio

```shell
sudo nano /etc/systemd/journal-remote.conf
```

```shell
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See journal-remote.conf(5) for details

[Remote]
Seal=false
SplitMode=host
# ServerKeyFile=/etc/ssl/private/journal-remote.pem
# ServerCertificateFile=/etc/ssl/certs/journal-remote.pem
# TrustedCertificateFile=/etc/ssl/ca/trusted.pem
```

### 3.5 Activación

```shell
sudo systemctl start systemd-journal-remote.service
```

### 3.6 Comprobaciones

Servicio funcionando:

![servicioup](https://i.imgur.com/XLy37Y4.png)

Vemos que se ha abierto el puerto necesario:

![puertoabierto](https://i.imgur.com/qL6MNm2.png)

## 4. Client

### 4.1 Paquetería

```shell
sudo apt update
sudo apt install systemd-journal-remote
```

### 4.2 Unidad systemd

```shell
sudo systemctl enable systemd-journal-upload.service
```

### 4.3 Configuración servicio

```shell
sudo nano /etc/systemd/journal-upload.conf
```

```shell
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See journal-upload.conf(5) for details

[Upload]
URL=http://10.0.0.2:19532
# ServerKeyFile=/etc/ssl/private/journal-upload.pem
# ServerCertificateFile=/etc/ssl/certs/journal-upload.pem
# TrustedCertificateFile=/etc/ssl/ca/trusted.pem
```

### 4.4 Activación

```shell
sudo systemctl restart systemd-journal-upload
```

## 5. Comprobaciones finales

En el servidor podemos comprobar que se ha creado un fichero en `/var/log/journal/remote` para el cliente:

```shell
vagrant@server:~$ ls -la /var/log/journal/remote
total 8200
drwxr-xr-x  2 systemd-journal-remote systemd-journal-remote    4096 Mar  7 04:13 .
drwxr-sr-x+ 4 root                   systemd-journal           4096 Mar  7 04:06 ..
-rw-r-----  1 systemd-journal-remote systemd-journal-remote 8388608 Mar  7 04:13 remote-10.0.0.3.journal
```

Esto funciona así por el modo `SplitMode=host`, y así separamos los ficheros de log por cliente.

Desde el cliente, para hacer pruebas de funcionamiento, podríamos mandar mensajes manualmente al log:

```shell
logger -p syslog.debug "### MENSAJE TEST DE ADRIAN ###"
```

Volviendo al servidor, podríamos leer el log del cliente y saltar al final para ver la prueba así:

```shell
sudo journalctl -e --file=/var/log/journal/remote/remote-10.0.0.3.journal
```

![pruebafinal](https://i.imgur.com/uBZfcq9.png)
