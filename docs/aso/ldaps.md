# LDAPs

## 1. Enunciado

Configura el servidor LDAP de alfa para que utilice el protocolo ldaps:// a la vez que el ldap:// utilizando el certificado x509 de la práctica de https o solicitando el correspondiente a través de gestiona. Realiza las modificaciones adecuadas en los clientes ldap de alfa para que todas las consultas se realicen por defecto utilizando ldaps://

## 2. Server

### 3.1 Paquetería

```shell
sudo apt update
sudo apt install systemd-journal-remote
```













































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
