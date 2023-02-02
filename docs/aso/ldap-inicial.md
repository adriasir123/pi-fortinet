# Práctica inicial

## 1. Enunciado

Realizar la instalación y configuración del servidor OpenLDAP en `server`, utilizando como base el nombre DNS asignado.  

Crear el usuario `prueba` y configurar `clientedebian` para que pueda validarse en `server` con dicho usuario.

El home de los usuarios deberá estar centralizado en `server`.

## 2. Escenario

```shell
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 2
    libvirt.memory = 1024
  end

  config.vm.define :server do |server|
    server.vm.box = "debian/bullseye64"
    server.vm.provider :libvirt do |server|
      server.memory = 3072
      server.cpus = 4
    end
    server.vm.hostname = "server"
    server.vm.network :private_network,
      :libvirt__network_name => "ldap-inicial",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientedebian do |clientedebian|
    clientedebian.vm.box = "debian/bullseye64"
    clientedebian.vm.hostname = "clientedebian"
    clientedebian.vm.network :private_network,
      :libvirt__network_name => "ldap-inicial",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

## 3. En `server`

### 3.1 Resoluciones

Dejo `/etc/hosts` así:

```shell
sudo nano /etc/hosts
```

```shell
127.0.0.1	localhost
127.0.0.2	bullseye
::1	        localhost ip6-localhost ip6-loopback
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters

127.0.1.1 server.adrianj.gonzalonazareno.org server
10.0.0.3 clientedebian.adrianj.gonzalonazareno.org clientedebian
```

Pruebo el FQDN:

```shell
vagrant@server:~$ hostname -f
server.adrianj.gonzalonazareno.org
```

Pruebo las resoluciones:

```shell
vagrant@server:~$ ping server.adrianj.gonzalonazareno.org -c 1
PING server.adrianj.gonzalonazareno.org (127.0.1.1) 56(84) bytes of data.
64 bytes from server.adrianj.gonzalonazareno.org (127.0.1.1): icmp_seq=1 ttl=64 time=0.129 ms

--- server.adrianj.gonzalonazareno.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.129/0.129/0.129/0.000 ms
vagrant@server:~$ ping clientedebian.adrianj.gonzalonazareno.org -c 1
PING clientedebian.adrianj.gonzalonazareno.org (10.0.0.3) 56(84) bytes of data.
64 bytes from clientedebian.adrianj.gonzalonazareno.org (10.0.0.3): icmp_seq=1 ttl=64 time=0.467 ms

--- clientedebian.adrianj.gonzalonazareno.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.467/0.467/0.467/0.000 ms
```

### 3.2 Paquetería

```shell
sudo apt update
sudo apt install slapd ldap-utils
```

Nos pide contraseña para el usuario `admin` *(1234)*:

![admin](https://i.imgur.com/klucS0s.png)

Muestro el estado inicial del directorio:

```shell
vagrant@server:~$ sudo slapcat
dn: dc=adrianj,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: adrianj.gonzalonazareno.org
dc: adrianj
structuralObjectClass: organization
entryUUID: 4b2a783c-3671-103d-94b3-f9f56aa6856a
creatorsName: cn=admin,dc=adrianj,dc=gonzalonazareno,dc=org
createTimestamp: 20230201114247Z
entryCSN: 20230201114247.462833Z#000000#000#000000
modifiersName: cn=admin,dc=adrianj,dc=gonzalonazareno,dc=org
modifyTimestamp: 20230201114247Z
```

### 3.3 Usuario

Genero una contraseña para el usuario `prueba` *(1234)*:

```shell
vagrant@server:~$ sudo slappasswd
New password:
Re-enter new password:
{SSHA}n/YjGnAXWJ6gDEIsZvVz69OFfLyMWbXo
```

Creo el siguiente fichero:

```shell
nano ldapuser.ldif
```

Con el siguiente contenido:

```shell
dn: uid=prueba,dc=adrianj,dc=gonzalonazareno,dc=org
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: prueba
sn: surname
userPassword: {SSHA}n/YjGnAXWJ6gDEIsZvVz69OFfLyMWbXo
loginShell: /bin/bash
uidNumber: 2000
gidNumber: 2000
homeDirectory: /srv/homes/prueba
```

Creo el usuario:

```shell
vagrant@server:~$ ldapadd -x -D cn=admin,dc=adrianj,dc=gonzalonazareno,dc=org -W -f ldapuser.ldif
Enter LDAP Password:
adding new entry "uid=prueba,dc=adrianj,dc=gonzalonazareno,dc=org"
```

Muestro que se ha creado:

```shell
vagrant@server:~$ ldapsearch -x -b "dc=adrianj,dc=gonzalonazareno,dc=org" -H ldap://server.adrianj.gonzalonazareno.org -D "cn=admin,dc=adrianj,dc=gonzalonazareno,dc=org" -W "objectclass=posixAccount"
Enter LDAP Password:
# extended LDIF
#
# LDAPv3
# base <dc=adrianj,dc=gonzalonazareno,dc=org> with scope subtree
# filter: objectclass=posixAccount
# requesting: ALL
#

# prueba, adrianj.gonzalonazareno.org
dn: uid=prueba,dc=adrianj,dc=gonzalonazareno,dc=org
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: prueba
sn: surname
userPassword:: e1NTSEF9bi9ZakduQVhXSjZnREVJc1p2Vno2OU9GZkx5TVdiWG8=
loginShell: /bin/bash
uidNumber: 2000
gidNumber: 2000
homeDirectory: /srv/homes/prueba
uid: prueba

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

### 3.4 NFS server

Creo el directorio que compartiremos donde se almacenarán todos los homes:

```shell
sudo mkdir /srv/homes
```

Instalo el paquete:

```shell
sudo apt install nfs-kernel-server
```

Defino el directorio compartido en el siguiente fichero:

```shell
sudo nano /etc/exports
```

```shell
/srv/homes 10.0.0.0/24(rw,sync,no_subtree_check,no_root_squash)
```

Reinicio nfs y actualizo la lista de directorios compartidos:

```shell
sudo systemctl restart nfs-kernel-server
sudo exportfs
```

## 4. En `clientedebian`

### 4.1 Resoluciones

Dejo `/etc/hosts` así:

```shell
sudo nano /etc/hosts
```

```shell
127.0.0.1	localhost
127.0.0.2	bullseye
::1	        localhost ip6-localhost ip6-loopback
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters

127.0.1.1 clientedebian.adrianj.gonzalonazareno.org clientedebian
10.0.0.2 server.adrianj.gonzalonazareno.org server
```

Pruebo el FQDN:

```shell
vagrant@clientedebian:~$ hostname -f
clientedebian.adrianj.gonzalonazareno.org
```

Pruebo las resoluciones:

```shell
vagrant@clientedebian:~$ ping clientedebian.adrianj.gonzalonazareno.org -c 1
PING clientedebian.adrianj.gonzalonazareno.org (127.0.1.1) 56(84) bytes of data.
64 bytes from clientedebian.adrianj.gonzalonazareno.org (127.0.1.1): icmp_seq=1 ttl=64 time=0.068 ms

--- clientedebian.adrianj.gonzalonazareno.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.068/0.068/0.068/0.000 ms
vagrant@clientedebian:~$ ping server.adrianj.gonzalonazareno.org -c 1
PING server.adrianj.gonzalonazareno.org (10.0.0.2) 56(84) bytes of data.
64 bytes from server.adrianj.gonzalonazareno.org (10.0.0.2): icmp_seq=1 ttl=64 time=0.244 ms

--- server.adrianj.gonzalonazareno.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.244/0.244/0.244/0.000 ms
```

### 4.2 Paquetería

```shell
sudo apt update
sudo apt install libnss-ldapd libpam-ldapd ldap-utils
```

![ldapuri](https://i.imgur.com/2TwIJ5n.png)

![ldapbase](https://i.imgur.com/1qd3DjA.png)

![ldapnameservices](https://i.imgur.com/kpCRrjv.png)

### 4.3 PAM

Añado la siguiente línea de configuración al final del fichero:

```shell
sudo nano /etc/pam.d/common-session
```

```shell
session required        pam_mkhomedir.so skel=/etc/skel umask=077
```

Reinicio los servicios

```shell
sudo systemctl restart nscd nslcd
```

### 4.4 NFS client

Creo el directorio donde se montará el compartido por el servidor:

```shell
sudo mkdir /srv/homes
```

Instalo el paquete:

```shell
sudo apt install nfs-common
```

Monto el directorio compartido:

```shell
sudo mount server.adrianj.gonzalonazareno.org:/srv/homes /srv/homes
```

Compruebo que se ha montado:

![remotomontado](https://i.imgur.com/8GosssR.png)

Hago el montaje permanente añadiendo la siguiente línea al final del fichero:

```shell
sudo nano /etc/fstab
```

```shell
server.adrianj.gonzalonazareno.org:/srv/homes   /srv/homes   nfs  defaults   0  0
```

### 4.5 Pruebas

Podemos logearnos con el usuario `prueba` remoto de LDAP de 2 maneras.

Así:

```shell
vagrant@clientedebian:~$ su - prueba
Password:
Creating directory '/srv/homes/prueba'.
prueba@clientedebian:~$
```

O así:

```shell
vagrant@clientedebian:~$ sudo login prueba
Password:
Linux clientedebian 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
prueba@clientedebian:~$
```

Pruebo que ese usuario no existe localmente, y era efectivamente un usuario remoto de LDAP:

```shell
vagrant@clientedebian:~$ cat /etc/passwd | grep prueba
vagrant@clientedebian:~$
```

Estando logeado, muestro el directorio home creado y creo un fichero y compruebo que aparece en el servidor:

![pruebafichero](https://i.imgur.com/i7XNngP.png)
