# Práctica inicial

Realiza la instalación y configuración básica de OpenLDAP en alfa,utilizando como base el nombre DNS asignado. Deberás crear un usuario llamado prueba y configurar una máquina cliente basada en Debian para que pueda validarse en servidor ldap configurado anteriormente con el usuario prueba.

## Escenario

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

`/etc/hosts`

```shell
127.0.1.1 server.adrianj.gonzalonazareno.org server
```




## Parte 1

> server ldap

```shell
sudo apt install slapd ldap-utils
```

Pide contraseña de admin:

![admin-passwd](https://i.imgur.com/A0R6dfu.png)

https://www.zytrax.com/books/ldap/ape/










## Parte 2

> Deberás crear un usuario llamado prueba

generate encrypted password 1234

```shell
vagrant@server:~$ sudo slappasswd
New password:
Re-enter new password:
{SSHA}l6/rFjEleZMfjBKIsf8ZvAAjDXlcKYrF
```

```shell
nano ldapuser.ldif
```

```shell
dn: uid=prueba,dc=adrianj,dc=gonzalonazareno,dc=org
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: prueba
sn: surname
userPassword: {SSHA}l6/rFjEleZMfjBKIsf8ZvAAjDXlcKYrF
loginShell: /bin/bash
uidNumber: 2000
gidNumber: 2000
homeDirectory: /srv/homes/prueba
```

```shell
vagrant@server:~$ ldapadd -x -D cn=admin,dc=adrianj,dc=gonzalonazareno,dc=org -W -f ldapuser.ldif
Enter LDAP Password:
adding new entry "uid=prueba,dc=adrianj,dc=gonzalonazareno,dc=org"
```








## Parte 3

> configuración de clientedebian

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



```shell
sudo apt update
sudo apt install libnss-ldapd libpam-ldapd ldap-utils
```

![ldapuri](https://i.postimg.cc/nV7hwzbx/ldap-client-uri.png)

![ldapbase](https://i.postimg.cc/zvfpvM1S/ldapbase.png)

![ldapnameservices](https://i.postimg.cc/BnhFQ6ny/ldapservices.png)





```shell
sudo nano /etc/pam.d/common-session
```

```shell
# add to the end if need (create home directory automatically at initial login)
session required        pam_mkhomedir.so skel=/etc/skel umask=077
```

```shell
sudo systemctl restart nscd nslcd
```



su - prueba
sudo login prueba






## Parte 4

> home compartido

NFS server:

```shell
sudo apt install nfs-kernel-server
sudo mkdir /srv/homes

sudo nano /etc/exports
/srv/homes 10.0.0.0/24(rw,sync,no_subtree_check,no_root_squash)

sudo systemctl restart nfs-kernel-server
sudo exportfs
```


NFS client:

```shell
sudo apt install nfs-common
sudo mkdir /srv/homes

sudo mount server.adrianj.gonzalonazareno.org:/srv/homes /srv/homes
```


```shell
Make the mount permanent
Using the root shell from "asmith", edit the file /etc/fstab on the client, adding a line to the end:
/etc/fstab (on nfsvm)
192.168.122.1:/home   /home   nfs  defaults   0  0
```





## Parte 5

> configuración de clienterocky

```shell
sudo vi /etc/hosts
```

```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
127.0.1.1 clienterocky.adrianj.gonzalonazareno.org clienterocky
10.0.0.2 server.adrianj.gonzalonazareno.org server
```



```shell
sudo dnf install openldap-clients sssd sssd-ldap sssd-tools oddjob-mkhomedir nscd

sudo mkdir /srv/homes
```


```shell
sudo authselect select sssd with-mkhomedir --force
```

```shell
sudo systemctl enable --now oddjobd.service
sudo systemctl status oddjobd.service
```




```shell
sudo vi /etc/openldap/ldap.conf

BASE    dc=adrianj,dc=gonzalonazareno,dc=org
URI     ldap://server.adrianj.gonzalonazareno.org/
TLS_REQCERT never
```




```shell
sudo vi /etc/sssd/sssd.conf


[domain/default]
id_provider = ldap
autofs_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldap://server.adrianj.gonzalonazareno.org/
ldap_search_base = dc=adrianj,dc=gonzalonazareno,dc=org
ldap_id_use_start_tls = false
ldap_tls_cacertdir = /etc/openldap/certs
cache_credentials = True
ldap_tls_reqcert = allow

[sssd]
services = nss, pam, autofs
domains = default

[nss]
homedir_substring = /srv/homes


sudo chmod 0600 /etc/sssd/sssd.conf

sudo systemctl restart sssd oddjobd
sudo systemctl enable sssd oddjobd

sudo systemctl status sssd
```











