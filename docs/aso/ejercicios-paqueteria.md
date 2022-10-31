# Ejercicios gestión de paquetería

## Escenario

```ruby
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 1
    libvirt.memory = 1024
  end

  config.vm.define :paqueteria do |paqueteria|
    paqueteria.vm.box = "debian/bullseye64"
    paqueteria.vm.hostname = "paqueteria"
  end

end
```

## Trabajo con apt, aptitude, dpkg

### Ejercicio 1 apt

> Explicar lo que sucede al hacer `sudo apt update`

`man apt` nos dice lo siguiente:

```shell
       update (apt-get(8))
           update is used to download package information from all configured sources. Other commands operate on this data to e.g. perform package upgrades
           or search in and display details about all packages available for installation.
```

Descarga información de los paquetes (package index) según nuestros sources configurados en el fichero `/etc/apt/sources.list` y en el directorio `/etc/apt/sources.list.d`.

Ejemplo de output:

```shell
vagrant@paqueteria:~$ sudo apt update
Get:1 https://deb.debian.org/debian bullseye InRelease [116 kB]
Get:2 https://security.debian.org/debian-security bullseye-security InRelease [48.4 kB]
Get:3 https://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]
Get:4 https://deb.debian.org/debian bullseye-backports InRelease [49.0 kB]
Get:5 https://deb.debian.org/debian bullseye/main Sources [8633 kB]
Get:6 https://security.debian.org/debian-security bullseye-security/main Sources [167 kB]
Get:7 https://security.debian.org/debian-security bullseye-security/main amd64 Packages [193 kB]
Get:8 https://security.debian.org/debian-security bullseye-security/main Translation-en [122 kB]
Get:9 https://deb.debian.org/debian bullseye/main amd64 Packages [8184 kB]
Get:10 https://deb.debian.org/debian bullseye/main Translation-en [6239 kB]
Get:11 https://deb.debian.org/debian bullseye-updates/main Sources [4812 B]
Get:12 https://deb.debian.org/debian bullseye-updates/main amd64 Packages [14.6 kB]
Get:13 https://deb.debian.org/debian bullseye-updates/main Translation-en [7929 B]
Get:14 https://deb.debian.org/debian bullseye-backports/main Sources [346 kB]
Get:15 https://deb.debian.org/debian bullseye-backports/main amd64 Packages [356 kB]
Get:16 https://deb.debian.org/debian bullseye-backports/main Translation-en [292 kB]
Fetched 24.8 MB in 12s (2068 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
18 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

> Explicar lo que sucede al hacer `sudo apt upgrade`

`man apt` nos dice lo siguiente:

```shell
       upgrade (apt-get(8))
           upgrade is used to install available upgrades of all packages currently installed on the system from the sources configured via sources.list(5).
           New packages will be installed if required to satisfy dependencies, but existing packages will never be removed. If an upgrade for a package
           requires the removal of an installed package the upgrade for this package isn't performed.
```

Instala las actualizaciones disponibles sobre los paquetes **actualmente instalados** en el sistema según nuestras definiciones en `/etc/apt/sources.list` y `/etc/apt/sources.list.d`.

Se instalarán nuevos paquetes si son necesarios para satisfacer dependencias, pero los paquetes existentes nunca serrán borrados.

Ejemplo de output:

```shell
vagrant@paqueteria:~$ sudo apt upgrade
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Run 'dpkg-reconfigure tzdata' if you wish to change it.

Setting up grub-common (2.06-4) ...
Setting up libdbus-1-3:amd64 (1.14.4-1) ...
Setting up dbus-session-bus-common (1.14.4-1) ...
Setting up isc-dhcp-common (4.4.3-P1-1) ...
Setting up dbus-system-bus-common (1.14.4-1) ...
Setting up dbus-bin (1.14.4-1) ...
Setting up bind9-libs:amd64 (1:9.18.4-2~bpo11+1) ...
Setting up grub2-common (2.06-4) ...
Setting up dbus-daemon (1.14.4-1) ...
Setting up grub-pc-bin (2.06-4) ...
Setting up grub-pc (2.06-4) ...
Replacing config file /etc/default/grub with new version
Installing for i386-pc platform.
Installation finished. No error reported.
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.19.0-0.deb11.2-amd64
Found initrd image: /boot/initrd.img-5.19.0-0.deb11.2-amd64
Found linux image: /boot/vmlinuz-5.10.0-19-amd64
Found initrd image: /boot/initrd.img-5.10.0-19-amd64
Found linux image: /boot/vmlinuz-5.10.0-18-amd64
Found initrd image: /boot/initrd.img-5.10.0-18-amd64
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done
Setting up dbus (1.14.4-1) ...
A reboot is required to replace the running dbus-daemon.
Please reboot the system when convenient.
dbus.service is a disabled or a static unit, not starting it.
Setting up bind9-host (1:9.18.4-2~bpo11+1) ...
Setting up bind9-dnsutils (1:9.18.4-2~bpo11+1) ...
Processing triggers for man-db (2.9.4-2) ...
Processing triggers for libc-bin (2.35-4) ...
```

### Ejercicio 2 apt

> Listar los paquetes que pueden ser actualizados y explicar el output

```shell
vagrant@paqueteria:~$ sudo apt list --upgradable
Listing... Done
bind9-dnsutils/stable-security 1:9.16.33-1~deb11u1 amd64 [upgradable from: 1:9.16.27-1~deb11u1]
bind9-host/stable-security 1:9.16.33-1~deb11u1 amd64 [upgradable from: 1:9.16.27-1~deb11u1]
bind9-libs/stable-security 1:9.16.33-1~deb11u1 amd64 [upgradable from: 1:9.16.27-1~deb11u1]
dbus/stable-security 1.12.24-0+deb11u1 amd64 [upgradable from: 1.12.20-2]
grub-common/stable-updates 2.06-3~deb11u2 amd64 [upgradable from: 2.06-3~deb11u1]
grub-pc-bin/stable-updates 2.06-3~deb11u2 amd64 [upgradable from: 2.06-3~deb11u1]
grub-pc/stable-updates 2.06-3~deb11u2 amd64 [upgradable from: 2.06-3~deb11u1]
grub2-common/stable-updates 2.06-3~deb11u2 amd64 [upgradable from: 2.06-3~deb11u1]
isc-dhcp-client/stable-security 4.4.1-2.3+deb11u1 amd64 [upgradable from: 4.4.1-2.3]
isc-dhcp-common/stable-security 4.4.1-2.3+deb11u1 amd64 [upgradable from: 4.4.1-2.3]
libc-bin/stable-updates 2.31-13+deb11u5 amd64 [upgradable from: 2.31-13+deb11u4]
libc-l10n/stable-updates 2.31-13+deb11u5 all [upgradable from: 2.31-13+deb11u4]
libc6/stable-updates 2.31-13+deb11u5 amd64 [upgradable from: 2.31-13+deb11u4]
libdbus-1-3/stable-security 1.12.24-0+deb11u1 amd64 [upgradable from: 1.12.20-2]
libexpat1/stable-security 2.2.10-2+deb11u5 amd64 [upgradable from: 2.2.10-2+deb11u3]
linux-image-amd64/stable-security 5.10.149-2 amd64 [upgradable from: 5.10.140-1]
locales/stable-updates 2.31-13+deb11u5 all [upgradable from: 2.31-13+deb11u4]
tzdata/stable-updates 2021a-1+deb11u7 all [upgradable from: 2021a-1+deb11u5]
```

Podemos discernir lo siguiente:

- Nombre de paquete
- Rama del repositorio
- Versión
- Arquitectura
- Versión actualmente instalada

### Ejercicio 3 apt

> Indicar sobre el paquete `openssh-client`:
>
>- Versión instalada
>- Versión candidata
>- Prioridad

![openssh-client](https://i.imgur.com/8o5AuDk.png)

### Ejercicio 4 apt

> Mostrar información de un paquete instalado o no

```shell
sudo apt show <paquete>
```

Ejemplo:

```shell
vagrant@paqueteria:~$ sudo apt show tree
Package: tree
Version: 1.8.0-1+b1
Priority: optional
Section: utils
Source: tree (1.8.0-1)
Maintainer: Florian Ernst <florian@debian.org>
Installed-Size: 118 kB
Depends: libc6 (>= 2.4)
Homepage: http://mama.indstate.edu/users/ice/tree/
Tag: implemented-in::c, interface::commandline, role::program,
 scope::utility, use::browsing, works-with::file
Download-Size: 49.6 kB
APT-Sources: https://deb.debian.org/debian bullseye/main amd64 Packages
Description: displays an indented directory tree, in color
 Tree is a recursive directory listing command that produces a depth indented
 listing of files, which is colorized ala dircolors if the LS_COLORS environment
 variable is set and output is to tty.
```

### Ejercicio 5 apt

> Mostrar toda la información que se pueda del paquete `openssh-client`

Podemos hacer:

```shell
vagrant@paqueteria:~$ sudo apt show openssh-client
Package: openssh-client
Version: 1:8.4p1-5+deb11u1
Priority: standard
Section: net
Source: openssh
Maintainer: Debian OpenSSH Maintainers <debian-ssh@lists.debian.org>
Installed-Size: 4401 kB
Provides: rsh-client, ssh-client
Depends: adduser (>= 3.10), dpkg (>= 1.7.0), passwd, libc6 (>= 2.26), libedit2 (>= 2.11-20080614-0), libfido2-1 (>= 1.5.0), libgssapi-krb5-2 (>= 1.17), libselinux1 (>= 3.1~), libssl1.1 (>= 1.1.1), zlib1g (>= 1:1.1.4)
Recommends: xauth
Suggests: keychain, libpam-ssh, monkeysphere, ssh-askpass
Conflicts: sftp
Breaks: openssh-sk-helper
Replaces: openssh-sk-helper, ssh, ssh-krb5
Homepage: http://www.openssh.com/
Tag: implemented-in::c, interface::commandline, interface::shell,
 network::client, protocol::sftp, protocol::ssh, role::program,
 security::authentication, security::cryptography, uitoolkit::ncurses,
 use::login, use::transmission, works-with::file
Download-Size: 929 kB
APT-Manual-Installed: yes
APT-Sources: https://deb.debian.org/debian bullseye/main amd64 Packages
Description: secure shell (SSH) client, for secure access to remote machines
 This is the portable version of OpenSSH, a free implementation of
 the Secure Shell protocol as specified by the IETF secsh working
 group.
 .
 Ssh (Secure Shell) is a program for logging into a remote machine
 and for executing commands on a remote machine.
 It provides secure encrypted communications between two untrusted
 hosts over an insecure network. X11 connections and arbitrary TCP/IP
 ports can also be forwarded over the secure channel.
 It can be used to provide applications with a secure communication
 channel.
 .
 This package provides the ssh, scp and sftp clients, the ssh-agent
 and ssh-add programs to make public key authentication more convenient,
 and the ssh-keygen, ssh-keyscan, ssh-copy-id and ssh-argv0 utilities.
 .
 In some countries it may be illegal to use any encryption at all
 without a special permit.
 .
 ssh replaces the insecure rsh, rcp and rlogin programs, which are
 obsolete for most purposes.
```

O también:

```shell
apt-cache showpkg openssh-client
```

El output es bastante largo, así que [entra en este gist](https://gist.github.com/adriasir123/75467379823af48957506b11db3b0c16) para verlo.

El último comando está muy bien, nos muestra información que no veríamos con `apt show`, pero a su vez carece de información que veríamos con `apt show` como el mantenedor/es del paquete.

### Ejercicio 6 apt

> Mostrar toda la información que se pueda del paquete candidato `openssh-client` a actualizar

Añado los repositorios de `unstable/main` para que `openssh-client` tenga un candidato más actualizado que el paquete actualmente instalado:

```shell
vagrant@paqueteria:~$ apt policy openssh-client
openssh-client:
  Installed: 1:8.4p1-5+deb11u1
  Candidate: 1:9.0p1-1+b2
  Version table:
     1:9.0p1-1+b2 500
        500 https://deb.debian.org/debian unstable/main amd64 Packages
 *** 1:8.4p1-5+deb11u1 500
        500 https://deb.debian.org/debian bullseye/main amd64 Packages
        100 /var/lib/dpkg/status
```

No existe un comando directo para hacer esto, así que podemos hacer:

```shell
sudo apt show openssh-client=$(apt-cache policy openssh-client | grep Candidate | awk '{print $2}')
```

Output:

```shell
vagrant@paqueteria:~$ sudo apt show openssh-client=$(apt-cache policy openssh-client | grep Candidate | awk '{print $2}')
Package: openssh-client
Version: 1:9.0p1-1+b2
Priority: standard
Section: net
Source: openssh (1:9.0p1-1)
Maintainer: Debian OpenSSH Maintainers <debian-ssh@lists.debian.org>
Installed-Size: 5772 kB
Provides: rsh-client, ssh-client
Depends: adduser (>= 3.10), dpkg (>= 1.7.0), passwd, libc6 (>= 2.34), libedit2 (>= 2.11-20080614-0), libfido2-1 (>= 1.8.0), libgssapi-krb5-2 (>= 1.17), libselinux1 (>= 3.1~), libssl3 (>= 3.0.5), zlib1g (>= 1:1.1.4)
Recommends: xauth
Suggests: keychain, libpam-ssh, monkeysphere, ssh-askpass
Conflicts: sftp
Breaks: openssh-sk-helper
Replaces: openssh-sk-helper, ssh, ssh-krb5
Homepage: http://www.openssh.com/
Tag: implemented-in::c, interface::commandline, interface::shell,
 network::client, protocol::sftp, protocol::ssh, role::program,
 security::authentication, security::cryptography, uitoolkit::ncurses,
 use::login, use::transmission, works-with::file
Download-Size: 973 kB
APT-Sources: https://deb.debian.org/debian unstable/main amd64 Packages
Description: secure shell (SSH) client, for secure access to remote machines
 This is the portable version of OpenSSH, a free implementation of
 the Secure Shell protocol as specified by the IETF secsh working
 group.
 .
 Ssh (Secure Shell) is a program for logging into a remote machine
 and for executing commands on a remote machine.
 It provides secure encrypted communications between two untrusted
 hosts over an insecure network. X11 connections and arbitrary TCP/IP
 ports can also be forwarded over the secure channel.
 It can be used to provide applications with a secure communication
 channel.
 .
 This package provides the ssh, scp and sftp clients, the ssh-agent
 and ssh-add programs to make public key authentication more convenient,
 and the ssh-keygen, ssh-keyscan, ssh-copy-id and ssh-argv0 utilities.
 .
 In some countries it may be illegal to use any encryption at all
 without a special permit.
 .
 ssh replaces the insecure rsh, rcp and rlogin programs, which are
 obsolete for most purposes.
```

### Ejercicio 7 apt

> Listar tanto con apt como con dpkg los contenidos del paquete `openssh-client` actual

Con `apt` sería así:

```shell
sudo apt install apt-file
sudo apt-file update
apt-file list openssh-client
```

Output:

```shell
vagrant@paqueteria:~$ apt-file list openssh-client
openssh-client: /etc/ssh/ssh_config
openssh-client: /usr/bin/scp
openssh-client: /usr/bin/sftp
openssh-client: /usr/bin/slogin
openssh-client: /usr/bin/ssh
openssh-client: /usr/bin/ssh-add
openssh-client: /usr/bin/ssh-agent
openssh-client: /usr/bin/ssh-argv0
openssh-client: /usr/bin/ssh-copy-id
openssh-client: /usr/bin/ssh-keygen
openssh-client: /usr/bin/ssh-keyscan
openssh-client: /usr/lib/openssh/agent-launch
openssh-client: /usr/lib/openssh/ssh-keysign
openssh-client: /usr/lib/openssh/ssh-pkcs11-helper
openssh-client: /usr/lib/openssh/ssh-sk-helper
openssh-client: /usr/lib/systemd/user/graphical-session-pre.target.wants/ssh-agent.service
openssh-client: /usr/lib/systemd/user/ssh-agent.service
openssh-client: /usr/share/apport/package-hooks/openssh-client.py
openssh-client: /usr/share/doc/openssh-client/NEWS.Debian.gz
openssh-client: /usr/share/doc/openssh-client/OVERVIEW.gz
openssh-client: /usr/share/doc/openssh-client/README
openssh-client: /usr/share/doc/openssh-client/README.Debian.gz
openssh-client: /usr/share/doc/openssh-client/README.dns
openssh-client: /usr/share/doc/openssh-client/README.tun.gz
openssh-client: /usr/share/doc/openssh-client/changelog.Debian.amd64.gz
openssh-client: /usr/share/doc/openssh-client/changelog.Debian.gz
openssh-client: /usr/share/doc/openssh-client/changelog.gz
openssh-client: /usr/share/doc/openssh-client/copyright
openssh-client: /usr/share/doc/openssh-client/faq.html
openssh-client: /usr/share/lintian/overrides/openssh-client
openssh-client: /usr/share/man/man1/scp.1.gz
openssh-client: /usr/share/man/man1/sftp.1.gz
openssh-client: /usr/share/man/man1/slogin.1.gz
openssh-client: /usr/share/man/man1/ssh-add.1.gz
openssh-client: /usr/share/man/man1/ssh-agent.1.gz
openssh-client: /usr/share/man/man1/ssh-argv0.1.gz
openssh-client: /usr/share/man/man1/ssh-copy-id.1.gz
openssh-client: /usr/share/man/man1/ssh-keygen.1.gz
openssh-client: /usr/share/man/man1/ssh-keyscan.1.gz
openssh-client: /usr/share/man/man1/ssh.1.gz
openssh-client: /usr/share/man/man5/ssh_config.5.gz
openssh-client: /usr/share/man/man8/ssh-keysign.8.gz
openssh-client: /usr/share/man/man8/ssh-pkcs11-helper.8.gz
openssh-client: /usr/share/man/man8/ssh-sk-helper.8.gz
```

Con `dpkg` sería así:

```shell
vagrant@paqueteria:~$ dpkg -L openssh-client
/.
/etc
/etc/ssh
/etc/ssh/ssh_config
/etc/ssh/ssh_config.d
/usr
/usr/bin
/usr/bin/scp
/usr/bin/sftp
/usr/bin/ssh
/usr/bin/ssh-add
/usr/bin/ssh-agent
/usr/bin/ssh-argv0
/usr/bin/ssh-copy-id
/usr/bin/ssh-keygen
/usr/bin/ssh-keyscan
/usr/lib
/usr/lib/openssh
/usr/lib/openssh/agent-launch
/usr/lib/openssh/ssh-keysign
/usr/lib/openssh/ssh-pkcs11-helper
/usr/lib/openssh/ssh-sk-helper
/usr/lib/systemd
/usr/lib/systemd/user
/usr/lib/systemd/user/graphical-session-pre.target.wants
/usr/lib/systemd/user/ssh-agent.service
/usr/share
/usr/share/apport
/usr/share/apport/package-hooks
/usr/share/apport/package-hooks/openssh-client.py
/usr/share/doc
/usr/share/doc/openssh-client
/usr/share/doc/openssh-client/NEWS.Debian.gz
/usr/share/doc/openssh-client/OVERVIEW.gz
/usr/share/doc/openssh-client/README
/usr/share/doc/openssh-client/README.Debian.gz
/usr/share/doc/openssh-client/README.dns
/usr/share/doc/openssh-client/README.tun.gz
/usr/share/doc/openssh-client/changelog.Debian.gz
/usr/share/doc/openssh-client/changelog.gz
/usr/share/doc/openssh-client/copyright
/usr/share/doc/openssh-client/faq.html
/usr/share/lintian
/usr/share/lintian/overrides
/usr/share/lintian/overrides/openssh-client
/usr/share/man
/usr/share/man/man1
/usr/share/man/man1/scp.1.gz
/usr/share/man/man1/sftp.1.gz
/usr/share/man/man1/ssh-add.1.gz
/usr/share/man/man1/ssh-agent.1.gz
/usr/share/man/man1/ssh-argv0.1.gz
/usr/share/man/man1/ssh-copy-id.1.gz
/usr/share/man/man1/ssh-keygen.1.gz
/usr/share/man/man1/ssh-keyscan.1.gz
/usr/share/man/man1/ssh.1.gz
/usr/share/man/man5
/usr/share/man/man5/ssh_config.5.gz
/usr/share/man/man8
/usr/share/man/man8/ssh-keysign.8.gz
/usr/share/man/man8/ssh-pkcs11-helper.8.gz
/usr/share/man/man8/ssh-sk-helper.8.gz
/usr/bin/slogin
/usr/lib/systemd/user/graphical-session-pre.target.wants/ssh-agent.service
/usr/share/man/man1/slogin.1.gz
```

### Ejercicio 8 apt

> Listar los contenidos de un paquete sin instalar ni descargar

```shell
apt-file list <paquete>
```

### Ejercicio 9 apt

> Simular la instalación de `openssh-client`

`man apt-get` nos dice:

```shell
       -s, --simulate, --just-print, --dry-run, --recon, --no-act
           No action; perform a simulation of events that would occur based on the current system state but do not actually change the system. Locking will
           be disabled (Debug::NoLocking) so the system state could change while apt-get is running. Simulations can also be executed by non-root users
           which might not have read access to all apt configuration distorting the simulation. A notice expressing this warning is also shown by default
           for non-root users (APT::Get::Show-User-Simulation-Note). Configuration Item: APT::Get::Simulate.

           Simulated runs print out a series of lines, each representing a dpkg operation: configure (Conf), remove (Remv) or unpack (Inst). Square
           brackets indicate broken packages, and empty square brackets indicate breaks that are of no consequence (rare).
```

Así que hacemos:

```shell
vagrant@paqueteria:~$ sudo apt install -s openssh-client
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libcbor0 libperl5.32
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libcbor0.8 libfido2-1 libssl3 openssh-server openssh-sftp-server runit-helper
Suggested packages:
  keychain libpam-ssh monkeysphere ssh-askpass molly-guard ufw
Recommended packages:
  xauth
The following NEW packages will be installed:
  libcbor0.8 libssl3
The following packages will be upgraded:
  libfido2-1 openssh-client openssh-server openssh-sftp-server runit-helper
5 upgraded, 2 newly installed, 0 to remove and 251 not upgraded.
Inst libcbor0.8 (0.8.0-2+b1 Debian:unstable [amd64])
Inst libssl3 (3.0.5-4 Debian:unstable [amd64])
Inst libfido2-1 [1.6.0-2] (1.12.0-1 Debian:unstable [amd64])
Inst openssh-sftp-server [1:8.4p1-5+deb11u1] (1:9.0p1-1+b2 Debian:unstable [amd64]) []
Inst openssh-server [1:8.4p1-5+deb11u1] (1:9.0p1-1+b2 Debian:unstable [amd64]) []
Inst openssh-client [1:8.4p1-5+deb11u1] (1:9.0p1-1+b2 Debian:unstable [amd64]) []
Inst runit-helper [2.10.3] (2.15.0 Debian:unstable [all])
Conf libcbor0.8 (0.8.0-2+b1 Debian:unstable [amd64])
Conf libssl3 (3.0.5-4 Debian:unstable [amd64])
Conf libfido2-1 (1.12.0-1 Debian:unstable [amd64])
Conf openssh-sftp-server (1:9.0p1-1+b2 Debian:unstable [amd64])
Conf openssh-server (1:9.0p1-1+b2 Debian:unstable [amd64])
Conf openssh-client (1:9.0p1-1+b2 Debian:unstable [amd64])
Conf runit-helper (2.15.0 Debian:unstable [all])
```

### Ejercicio 10 apt

> Mostrar comando que liste los bugs de un paquete

```shell
sudo apt install apt-listbugs
sudo apt-listbugs -s all list <paquete>
```

Ejemplo:

```shell
vagrant@paqueteria:~$ sudo apt-listbugs -s all list tree
Retrieving bug reports... Done
Parsing Found/Fixed information... Done
normal bugs of tree (→ ) <Forwarded>
 b1 - #934520 - tree: --filefrom option doesn't recognize symbolic links
minor bugs of tree (→ ) <Forwarded>
 b2 - #1014621 - tree: --timefmt doesn't imply -D, manual says it should
wishlist bugs of tree (→ ) <Forwarded>
 b3 - #865867 - make tree -H "mobile friendly"
 b4 - #939644 - Viewport not set... need way to add HTML headers
Summary:
 tree(4 bugs)
```

### Ejercicio 11 apt

> Realizar una actualización completa de la paquetería

```shell
sudo apt update && sudo apt upgrade
```

> Actualizar los paquetes que contienen la cadena openssh

```shell
sudo apt upgrade *openssh*
```

El proceso es largo, así que [mostraré como ejemplo](https://gist.github.com/adriasir123/bfd22429bf7da069b65b4d049119812c) lo que aparece antes de confirmar.

> Hacer lo mismo pero con un for en bash

Tenemos que ejecutar el siguiente script con sudo:

```shell
#!/bin/bash

pkgnames=$(apt-cache pkgnames openssh)

for i in $pkgnames
do
   apt upgrade $i
done
```

> Hacer lo mismo pero con xargs

```shell
apt-cache pkgnames openssh | xargs sudo apt upgrade -y
```

### Ejercicio 12 apt

> Mostrar las dependencias inversas de un paquete

`man apt-cache` nos dice:

```shell
       rdepends pkg...
           rdepends shows a listing of each reverse dependency a package has.
```

Así que hacemos:

```shell
apt-cache rdepends <paquete>
```

Ejemplo:

```shell
vagrant@paqueteria:~$ apt-cache rdepends tree
tree
Reverse Depends:
  pass
  sisu-complete
  sisu
  pypass
  progress-linux-desktop
  pass
  mle
  inxi
  hollywood
  gopass
  creddump7
  bfh-desktop
  inxi
  sisu-complete
  sisu
  pypass
  progress-linux-desktop
  gopass
  mle
  keyringer
  inxi
  hollywood
```

### Ejercicio 13 apt

> Mostrar el paquete al que pertenece un fichero

`man dpkg` nos dice:

```shell
           -S, --search filename-search-pattern...
               Search for a filename from installed packages.
```

Así que hacemos:

```shell
dpkg -S <ruta-al-binario>
```

Ejemplo:

```shell
vagrant@paqueteria:~$ dpkg -S /bin/ls
coreutils: /bin/ls
```

### Ejercicio 14 apt

> Liberar la caché de la paquetería

`man apt-get` nos dice:

```shell
       clean
           clean clears out the local repository of retrieved package files. It removes everything but the lock file from /var/cache/apt/archives/ and
           /var/cache/apt/archives/partial/.
```

Así que hacemos:

```shell
sudo apt clean
```

### Ejercicio 15 apt

> Instalar el paquete `keyboard-configuration` automáticamente pasando parámetros con debconf

Instalo `debconf-utils`:

```shell
sudo apt install debconf-utils
```

Instalo una primera vez `keyboard-configuration` sin pasarle parámetros para poder obtener su configuración:

```shell
sudo apt install keyboard-configuration
```

Exporto las "selections" a un fichero:

```shell
sudo debconf-get-selections | grep keyboard-configuration > selections.conf
```

Desinstalo `keyboard-configuration`:

```shell
sudo apt purge keyboard-configuration
```

Paso los parámetros del fichero a debconf:

```shell
sudo debconf-set-selections < selections.conf
```

Al volver a instalar el paquete, los parámetros evitarán las preguntas de debconf:

![keyboardautomatico](https://i.postimg.cc/jSTT3xD7/keyboardnopreguntas.gif)

### Ejercicio 16 apt

> Mostrar la configuración de locales actual

```shell
vagrant@paqueteria:~$ localectl
   System Locale: LANG=C.UTF-8
       VC Keymap: n/a
      X11 Layout: es
       X11 Model: pc105
```

> Reconfigurar el paquete locales, añadiendo una nueva locale

```shell
sudo dpkg-reconfigure locales
```

![añadolocales](https://i.imgur.com/7oHofdu.png)

![defaultlocale](https://i.imgur.com/I2y3vgO.png)

Compruebo que ha funcionado:

```shell
vagrant@paqueteria:~$ localectl
   System Locale: LANG=es_ES.UTF-8
       VC Keymap: n/a
      X11 Layout: es
       X11 Model: pc105
```

Existen algunas otras variables que podemos usar:

```shell
export LANGUAGE=es_ES.UTF-8
export LANG=es_ES.UTF-8
export LC_ALL=es_ES.UTF-8
```

### Ejercicio 17 apt

> Interrumpir la instalación de un paquete y explicar los pasos para continuar la instalación

Para crear esta situación puedo forzar el apagado de la máquina vagrant durante la instalación de `mariadb-server`:

![forcedfailmariadb](https://i.postimg.cc/0QCFj5zm/forcefailmariadb.gif)

Si volvemos a levantar la máquina e intentamos seguir con la instalación, nos encontramos con el siguiente error:

```shell
vagrant@paqueteria:~$ sudo apt install mariadb-server
E: dpkg was interrupted, you must manually run 'sudo dpkg --configure -a' to correct the problem.
```

Hacemos lo que nos dice:

```shell
vagrant@paqueteria:~$ sudo dpkg --configure -a
Setting up libconfig-inifiles-perl (3.000003-1) ...
Setting up galera-4 (26.4.11-0+deb11u1) ...
Setting up gawk (1:5.1.0-1) ...
Setting up psmisc (23.4-2) ...
Setting up libsnappy1v5:amd64 (1.1.8-1) ...
Setting up socat (1.7.4.1-3) ...
Setting up libmariadb3:amd64 (1:10.5.15-0+deb11u1) ...
Setting up libdbi-perl:amd64 (1.643-3+b1) ...
Setting up rsync (3.2.3-4+deb11u1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rsync.service → /lib/systemd/system/rsync.service.
Setting up mariadb-server-core-10.5 (1:10.5.15-0+deb11u1) ...
Setting up mariadb-client-core-10.5 (1:10.5.15-0+deb11u1) ...
Setting up mariadb-client-10.5 (1:10.5.15-0+deb11u1) ...
Processing triggers for man-db (2.9.4-2) ...
Processing triggers for libc-bin (2.31-13+deb11u4) ...
```

Ya podríamos volver a instalar:

```shell
vagrant@paqueteria:~$ sudo apt install mariadb-server
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  galera-4 gawk libcgi-fast-perl libcgi-pm-perl libclone-perl libconfig-inifiles-perl libdbd-mariadb-perl libdbi-perl libencode-locale-perl libfcgi-bin
  libfcgi-perl libfcgi0ldbl libhtml-parser-perl libhtml-tagset-perl libhtml-template-perl libhttp-date-perl libhttp-message-perl libio-html-perl
  liblwp-mediatypes-perl libmariadb3 libmpfr6 libsigsegv2 libsnappy1v5 libterm-readkey-perl libtimedate-perl liburi-perl mariadb-client-10.5
  mariadb-client-core-10.5 mariadb-common mariadb-server-10.5 mariadb-server-core-10.5 mysql-common psmisc rsync socat
Suggested packages:
  gawk-doc libmldbm-perl libnet-daemon-perl libsql-statement-perl libdata-dump-perl libipc-sharedcache-perl libwww-perl mailx mariadb-test netcat-openbsd
The following NEW packages will be installed:
  galera-4 gawk libcgi-fast-perl libcgi-pm-perl libclone-perl libconfig-inifiles-perl libdbd-mariadb-perl libdbi-perl libencode-locale-perl libfcgi-bin
  libfcgi-perl libfcgi0ldbl libhtml-parser-perl libhtml-tagset-perl libhtml-template-perl libhttp-date-perl libhttp-message-perl libio-html-perl
  liblwp-mediatypes-perl libmariadb3 libmpfr6 libsigsegv2 libsnappy1v5 libterm-readkey-perl libtimedate-perl liburi-perl mariadb-client-10.5
  mariadb-client-core-10.5 mariadb-common mariadb-server mariadb-server-10.5 mariadb-server-core-10.5 mysql-common psmisc rsync socat
0 upgraded, 36 newly installed, 0 to remove and 18 not upgraded.
Need to get 19.7 MB of archives.
After this operation, 162 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://deb.debian.org/debian bullseye/main amd64 libmpfr6 amd64 4.1.0-3 [2012 kB]
Get:2 https://deb.debian.org/debian bullseye/main amd64 libsigsegv2 amd64 2.13-1 [34.8 kB]
Get:3 https://deb.debian.org/debian bullseye/main amd64 gawk amd64 1:5.1.0-1 [605 kB]
Get:4 https://deb.debian.org/debian bullseye/main amd64 mysql-common all 5.8+1.0.7 [7464 B]
Get:5 https://deb.debian.org/debian bullseye/main amd64 mariadb-common all 1:10.5.15-0+deb11u1 [36.7 kB]
Get:6 https://deb.debian.org/debian bullseye/main amd64 galera-4 amd64 26.4.11-0+deb11u1 [804 kB]
Get:7 https://deb.debian.org/debian bullseye/main amd64 libdbi-perl amd64 1.643-3+b1 [780 kB]
Get:8 https://deb.debian.org/debian bullseye/main amd64 libconfig-inifiles-perl all 3.000003-1 [52.1 kB]
Get:9 https://deb.debian.org/debian bullseye/main amd64 libmariadb3 amd64 1:10.5.15-0+deb11u1 [176 kB]
Get:10 https://deb.debian.org/debian bullseye/main amd64 mariadb-client-core-10.5 amd64 1:10.5.15-0+deb11u1 [783 kB]
Get:11 https://deb.debian.org/debian bullseye/main amd64 mariadb-client-10.5 amd64 1:10.5.15-0+deb11u1 [1509 kB]
Get:12 https://deb.debian.org/debian bullseye/main amd64 libsnappy1v5 amd64 1.1.8-1 [17.9 kB]
Get:13 https://deb.debian.org/debian bullseye/main amd64 mariadb-server-core-10.5 amd64 1:10.5.15-0+deb11u1 [6689 kB]
Get:14 https://deb.debian.org/debian bullseye/main amd64 psmisc amd64 23.4-2 [198 kB]
Get:15 https://deb.debian.org/debian bullseye/main amd64 rsync amd64 3.2.3-4+deb11u1 [396 kB]
Get:16 https://deb.debian.org/debian bullseye/main amd64 socat amd64 1.7.4.1-3 [370 kB]
Get:17 https://deb.debian.org/debian bullseye/main amd64 mariadb-server-10.5 amd64 1:10.5.15-0+deb11u1 [4260 kB]
Get:18 https://deb.debian.org/debian bullseye/main amd64 libhtml-tagset-perl all 3.20-4 [13.0 kB]
  mariadb-server
The following packages will be upgraded:
  mariadb-server-10.5
1 upgraded, 1 newly installed, 0 to remove and 18 not upgraded.
1 not fully installed or removed.
Need to get 0 B/4295 kB of archives.
After this operation, 66.9 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Reading changelogs... Done
Preconfiguring packages ...
(Reading database ... 25893 files and directories currently installed.)
Preparing to unpack .../mariadb-server-10.5_1%3a10.5.15-0+deb11u1_amd64.deb ...
Unpacking mariadb-server-10.5 (1:10.5.15-0+deb11u1) over (1:10.5.15-0+deb11u1) ...
Selecting previously unselected package mariadb-server.
Preparing to unpack .../mariadb-server_1%3a10.5.15-0+deb11u1_all.deb ...
Unpacking mariadb-server (1:10.5.15-0+deb11u1) ...
Setting up mariadb-server-10.5 (1:10.5.15-0+deb11u1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /lib/systemd/system/mariadb.service.
Setting up mariadb-server (1:10.5.15-0+deb11u1) ...
Processing triggers for man-db (2.9.4-2) ...
```

### Ejercicio 18 apt

> Actualizar todos los paquetes de manera no interactiva

`man apt-get` nos dice:

```shell
       -y, --yes, --assume-yes
           Automatic yes to prompts; assume "yes" as answer to all prompts and run non-interactively. If an undesirable situation, such as changing a held
           package, trying to install an unauthenticated package or removing an essential package occurs then apt-get will abort. Configuration Item:
           APT::Get::Assume-Yes.
```

Así que tenemos 2 maneras:

```shell
sudo apt update && sudo apt upgrade -y
sudo apt update && sudo apt dist-upgrade -y
```

La primera manera conlleva menos riesgo que la segunda, y lo único que podría parar la "no interactividad" sería un menú de debconf u otras situaciones excepcionales.

[Ejemplo de uso de la primera manera](https://gist.github.com/adriasir123/004ec6396a998c4edd9d57d3a2fdd8af)

[Ejemplo de uso de la segunda manera](https://gist.github.com/adriasir123/9c3b7c4e356851dce9f0c6bcc942f04e)

### Ejercicio 19 apt

> Bloquear la actualización de ciertos paquetes

`man apt-mark` nos dice:

```shell
       hold
           hold is used to mark a package as held back, which will prevent the package from being automatically installed, upgraded or removed.
```

Así que hacemos:

```shell
sudo apt-mark hold <paquete>
```

Ejemplo:

```shell
vagrant@paqueteria:~$ sudo apt-mark hold tree
tree set on hold.
```

Podemos mostrar la lista de paquetes que tenemos en "hold":

```shell
vagrant@paqueteria:~$ apt-mark showhold
tree
```

## Trabajo con ficheros .deb

### Ejercicio 1 deb

> Descargar un paquete sin instalarlo

`man apt-get` nos dice:

```shell
       -d, --download-only
           Download only; package files are only retrieved, not unpacked or installed. Configuration Item: APT::Get::Download-Only.
```

Así que hacemos:

```shell
vagrant@paqueteria:~$ sudo apt -d install tree
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  tree
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 49.6 kB of archives.
After this operation, 118 kB of additional disk space will be used.
Get:1 https://deb.debian.org/debian bullseye/main amd64 tree amd64 1.8.0-1+b1 [49.6 kB]
Fetched 49.6 kB in 0s (297 kB/s)
Download complete and in download only mode
```

El mensaje `Download complete and in download only mode` es claro.

Podemos comprobar que efectivamente no se ha instalado:

```shell
vagrant@paqueteria:~$ apt policy tree
tree:
  Installed: (none)
  Candidate: 1.8.0-1+b1
  Version table:
     1.8.0-1+b1 500
        500 https://deb.debian.org/debian bullseye/main amd64 Packages
```

Podemos comprobar por último que tenemos el paquete descargado:

```shell
vagrant@paqueteria:~$ ls -la /var/cache/apt/archives
total 64
drwxr-xr-x 3 root root  4096 Oct 30 09:58 .
drwxr-xr-x 3 root root  4096 Oct 30 09:51 ..
-rw-r----- 1 root root     0 Oct 29 19:39 lock
drwx------ 2 _apt root  4096 Oct 30 09:58 partial
-rw-r--r-- 1 root root 49636 Aug  6  2019 tree_1.8.0-1+b1_amd64.deb
```

Existe otra manera de hacer esto, y es con:

```shell
vagrant@paqueteria:~$ apt download tree
Get:1 https://deb.debian.org/debian bullseye/main amd64 tree amd64 1.8.0-1+b1 [49.6 kB]
Fetched 49.6 kB in 0s (214 kB/s)
```

Pero hay notables diferencias:

- No necesitamos sudo
- Descarga los .deb en el directorio actual

```shell
vagrant@paqueteria:~$ ls -la
total 84
drwxr-xr-x 4 vagrant vagrant  4096 Oct 30 10:16 .
drwxr-xr-x 3 root    root     4096 Sep 12 05:17 ..
-rw-r--r-- 1 vagrant vagrant   220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 vagrant vagrant  3526 Mar 27  2022 .bashrc
-rw------- 1 vagrant vagrant   125 Oct 30 10:11 .lesshst
drwxr-xr-x 3 vagrant vagrant  4096 Oct 29 19:34 .local
-rw-r--r-- 1 vagrant vagrant   807 Mar 27  2022 .profile
drwx------ 2 vagrant vagrant  4096 Oct 29 15:19 .ssh
-rw-r--r-- 1 vagrant vagrant 49636 Aug  6  2019 tree_1.8.0-1+b1_amd64.deb
```

### Ejercicio 2 deb

> Listar el contenido de un paquete deb

`man dpkg` nos dice:

```shell
           -c, --contents archive
               List contents of a deb package.
```

Así que para trabajar directamente con paquetes .deb, podemos hacer:

```shell
vagrant@paqueteria:~$ dpkg -c tree_1.8.0-1+b1_amd64.deb
drwxr-xr-x root/root         0 2019-08-06 19:31 ./
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/bin/
-rwxr-xr-x root/root     81512 2019-08-06 19:31 ./usr/bin/tree
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/doc/
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/doc/tree/
-rw-r--r-- root/root      2723 2015-06-22 20:17 ./usr/share/doc/tree/README.gz
-rw-r--r-- root/root      2403 2015-02-11 20:57 ./usr/share/doc/tree/TODO
-rw-r--r-- root/root       210 2019-08-06 19:31 ./usr/share/doc/tree/changelog.Debian.amd64.gz
-rw-r--r-- root/root      4294 2019-08-06 19:31 ./usr/share/doc/tree/changelog.Debian.gz
-rw-r--r-- root/root      5276 2018-11-16 15:12 ./usr/share/doc/tree/changelog.gz
-rw-r--r-- root/root      2441 2018-12-11 10:19 ./usr/share/doc/tree/copyright
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/man/
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/man/man1/
-rw-r--r-- root/root      4627 2019-08-06 19:31 ./usr/share/man/man1/tree.1.gz
```

Si queremos trabajar con nombres de paquetes en vez de con ficheros de paquetes, podemos hacer:

```shell
vagrant@paqueteria:~$ sudo apt-file update
Hit:1 https://deb.debian.org/debian bullseye InRelease
Get:2 https://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]
Get:3 https://deb.debian.org/debian bullseye-backports InRelease [49.0 kB]
Get:4 https://security.debian.org/debian-security bullseye-security InRelease [48.4 kB]
Get:5 https://deb.debian.org/debian bullseye/main all Contents (deb) [31.0 MB]
Get:6 https://security.debian.org/debian-security bullseye-security/main Sources [167 kB]
Get:7 https://security.debian.org/debian-security bullseye-security/main amd64 Packages [193 kB]
Get:8 https://security.debian.org/debian-security bullseye-security/main Translation-en [122 kB]
Get:9 https://deb.debian.org/debian bullseye/main amd64 Contents (deb) [10.3 MB]
Get:10 https://deb.debian.org/debian bullseye-updates/main amd64 Contents (deb) [68.7 kB]
Get:11 https://deb.debian.org/debian bullseye-updates/main all Contents (deb) [25.0 kB]
Get:12 https://deb.debian.org/debian bullseye-backports/main all Contents (deb) [4428 kB]
Get:13 https://deb.debian.org/debian bullseye-backports/main amd64 Contents (deb) [1139 kB]
Fetched 47.6 MB in 13s (3732 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
vagrant@paqueteria:~$ apt-file list tree
tree: /usr/bin/tree
tree: /usr/share/doc/tree/README.gz
tree: /usr/share/doc/tree/TODO
tree: /usr/share/doc/tree/changelog.Debian.amd64.gz
tree: /usr/share/doc/tree/changelog.Debian.gz
tree: /usr/share/doc/tree/changelog.gz
tree: /usr/share/doc/tree/copyright
tree: /usr/share/man/man1/tree.1.gz
```

### Ejercicio 3 deb

#### 3.1

> Descargar el paquete que nos proveerá el binario `ar`

```shell
sudo apt install binutils
```

#### 3.2

> Listar el contenido del paquete .deb descargado

`ar --help` nos dice:

```shell
  t[O][v]      - display contents of the archive
```

Así que hacemos:

```shell
vagrant@paqueteria:~$ ar tOv tree_1.8.0-1+b1_amd64.deb
rw-r--r-- 0/0      4 Aug  6 19:31 2019 debian-binary 0x44
rw-r--r-- 0/0    844 Aug  6 19:31 2019 control.tar.xz 0x84
rw-r--r-- 0/0  48600 Aug  6 19:31 2019 data.tar.xz 0x40c
```

#### 3.3

> Extraer el paquete

`ar --help` nos dice:

```shell
  x[o]         - extract file(s) from the archive
```

Así que primero creo un directorio para la extración y muevo el .deb:

```shell
mkdir tree-extraction && mv tree_1.8.0-1+b1_amd64.deb tree-extraction
cd tree-extraction
```

Aquí, extraigo:

```shell
ar x tree_1.8.0-1+b1_amd64.deb
```

Muestro lo extraído:

```shell
vagrant@paqueteria:~/tree-extraction$ ls -la
total 116
drwxr-xr-x 2 vagrant vagrant  4096 Oct 30 11:25 .
drwxr-xr-x 5 vagrant vagrant  4096 Oct 30 11:22 ..
-rw-r--r-- 1 vagrant vagrant   844 Oct 30 11:25 control.tar.xz
-rw-r--r-- 1 vagrant vagrant 48600 Oct 30 11:25 data.tar.xz
-rw-r--r-- 1 vagrant vagrant     4 Oct 30 11:25 debian-binary
-rw-r--r-- 1 vagrant vagrant 49636 Aug  6  2019 tree_1.8.0-1+b1_amd64.deb
```

> Indicar la función de cada fichero extraído

**debian-binary**

Es un fichero que contiene una única línea con el número de versión del formato del paquete:

```shell
vagrant@paqueteria:~/tree-extraction$ cat debian-binary
2.0
```

**control.tar.xz**

Es un archivo tar comprimido que puede contener:

- Scripts de los mantenedores: sirven para ayudar en la instalación, actualización, o borrado de los paquetes
- Metadatos: como el nombre del paquete, su versión, dependencias y mantenedor
- Checksums

```shell
vagrant@paqueteria:~/tree-extraction$ tar -tvf control.tar.xz
drwxr-xr-x root/root         0 2019-08-06 19:31 ./
-rw-r--r-- root/root       510 2019-08-06 19:31 ./control
-rw-r--r-- root/root       512 2019-08-06 19:31 ./md5sums
```

Podemos extraerlo y mostrar sus contenidos para tenerlo todo más claro:

```shell
vagrant@paqueteria:~/tree-extraction$ tar -xvf control.tar.xz
./
./control
./md5sums
vagrant@paqueteria:~/tree-extraction$ cat control
Package: tree
Source: tree (1.8.0-1)
Version: 1.8.0-1+b1
Architecture: amd64
Maintainer: Florian Ernst <florian@debian.org>
Installed-Size: 115
Depends: libc6 (>= 2.4)
Section: utils
Priority: optional
Homepage: http://mama.indstate.edu/users/ice/tree/
Description: displays an indented directory tree, in color
 Tree is a recursive directory listing command that produces a depth indented
 listing of files, which is colorized ala dircolors if the LS_COLORS environment
 variable is set and output is to tty.
vagrant@paqueteria:~/tree-extraction$ cat md5sums
8dab5530d608f37d948b8fd6271e2ecb  usr/bin/tree
f49004c1057dfeaed3db1dceb050ac5d  usr/share/doc/tree/README.gz
3eb175af3a5716275b668ce860c2bfad  usr/share/doc/tree/TODO
79d76b44e3ea99e40e3800d02b02ac1b  usr/share/doc/tree/changelog.Debian.amd64.gz
039344412ac0fc0d1528c9ff3803941e  usr/share/doc/tree/changelog.Debian.gz
25d6d6da4b1e1bf2041c9b022367d07a  usr/share/doc/tree/changelog.gz
e51a3c84c3b33325acc683347bfb4d3f  usr/share/doc/tree/copyright
e0fceaec1e51e1da23fce6741a21e5a4  usr/share/man/man1/tree.1.gz
```

**data.tar.xz**

Es un archivo tar comprimido que contiene los ficheros que se instalarán:

```shell
vagrant@paqueteria:~/tree-extraction$ tar -tvf data.tar.xz
drwxr-xr-x root/root         0 2019-08-06 19:31 ./
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/bin/
-rwxr-xr-x root/root     81512 2019-08-06 19:31 ./usr/bin/tree
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/doc/
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/doc/tree/
-rw-r--r-- root/root      2723 2015-06-22 20:17 ./usr/share/doc/tree/README.gz
-rw-r--r-- root/root      2403 2015-02-11 20:57 ./usr/share/doc/tree/TODO
-rw-r--r-- root/root       210 2019-08-06 19:31 ./usr/share/doc/tree/changelog.Debian.amd64.gz
-rw-r--r-- root/root      4294 2019-08-06 19:31 ./usr/share/doc/tree/changelog.Debian.gz
-rw-r--r-- root/root      5276 2018-11-16 15:12 ./usr/share/doc/tree/changelog.gz
-rw-r--r-- root/root      2441 2018-12-11 10:19 ./usr/share/doc/tree/copyright
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/man/
drwxr-xr-x root/root         0 2019-08-06 19:31 ./usr/share/man/man1/
-rw-r--r-- root/root      4627 2019-08-06 19:31 ./usr/share/man/man1/tree.1.gz
```

Podemos extraerlo y mostrar sus contenidos para tenerlo todo más claro:

```shell
vagrant@paqueteria:~/tree-extraction$ mkdir data
vagrant@paqueteria:~/tree-extraction$ tar -xvf data.tar.xz -C data
./
./usr/
./usr/bin/
./usr/bin/tree
./usr/share/
./usr/share/doc/
./usr/share/doc/tree/
./usr/share/doc/tree/README.gz
./usr/share/doc/tree/TODO
./usr/share/doc/tree/changelog.Debian.amd64.gz
./usr/share/doc/tree/changelog.Debian.gz
./usr/share/doc/tree/changelog.gz
./usr/share/doc/tree/copyright
./usr/share/man/
./usr/share/man/man1/
./usr/share/man/man1/tree.1.gz
vagrant@paqueteria:~/tree-extraction$ tree data
data
└── usr
    ├── bin
    │   └── tree
    └── share
        ├── doc
        │   └── tree
        │       ├── README.gz
        │       ├── TODO
        │       ├── changelog.Debian.amd64.gz
        │       ├── changelog.Debian.gz
        │       ├── changelog.gz
        │       └── copyright
        └── man
            └── man1
                └── tree.1.gz

7 directories, 8 files
```

Vemos que tenemos el binario, las páginas man...etc.

## Trabajo con repositorios

### Ejercicio 1 repos

> Añadir a `sources.list` los repositorios bullseye-backports y sid

Al estar usando una máquina Vagrant los repositorios de `bullseye-backports` los tengo ya, y sólo tendría que añadir los de sid.

Dejo `/etc/apt/sources.list` de la siguiente manera:

```shell
vagrant@paqueteria:~$ cat /etc/apt/sources.list
deb https://deb.debian.org/debian bullseye main
deb-src https://deb.debian.org/debian bullseye main

deb https://security.debian.org/debian-security bullseye-security main
deb-src https://security.debian.org/debian-security bullseye-security main

deb https://deb.debian.org/debian bullseye-updates main
deb-src https://deb.debian.org/debian bullseye-updates main

# Backports

deb https://deb.debian.org/debian bullseye-backports main
deb-src https://deb.debian.org/debian bullseye-backports main

# Unstable

deb https://deb.debian.org/debian unstable main
deb-src https://deb.debian.org/debian unstable main
```

Si queremos que estos cambios se apliquen, tenemos que hacer:

```shell
sudo apt update
```

### Ejercicio 2 repos

> Configurar apt para que los paquetes de bullseye tengan mayor prioridad

Creo el fichero `/etc/apt/preferences`, que no existe por defecto:

```shell
sudo nano /etc/apt/preferences
```

Con el siguiente contenido:

```shell
Package: *
Pin: release n=bullseye
Pin-Priority: 600
```

Hago la prueba con un paquete para ver cuál sería el candidato y mostrar de paso las prioridades cambiadas:

```shell
vagrant@paqueteria:/etc/apt$ apt policy htop
htop:
  Installed: (none)
  Candidate: 3.0.5-7
  Version table:
     3.2.1-1 500
        500 https://deb.debian.org/debian unstable/main amd64 Packages
     3.0.5-7 600
        600 https://deb.debian.org/debian bullseye/main amd64 Packages
```

### Ejercicio 3 repos

> Configurar apt para que los paquetes de bullseye-backports tengan mayor prioridad que los de unstable

Si no tocamos nada, vemos que por defecto unstable tiene mayor prioridad que bullseye-backports:

```shell
vagrant@paqueteria:~$ apt policy 7zip
7zip:
  Installed: (none)
  Candidate: 22.01+dfsg-4
  Version table:
     22.01+dfsg-4 500
        500 https://deb.debian.org/debian unstable/main amd64 Packages
     22.01+dfsg-2~bpo11+1 100
        100 https://deb.debian.org/debian bullseye-backports/main amd64 Packages
```

Así que para modificarlo, dejo `/etc/apt/preferences` de la siguiente manera:

```shell
Package: *
Pin: release n=bullseye
Pin-Priority: 600

Package: *
Pin: release n=bullseye-backports
Pin-Priority: 550
```

Vemos los cambios:

```shell
vagrant@paqueteria:~$ apt policy 7zip
7zip:
  Installed: (none)
  Candidate: 22.01+dfsg-2~bpo11+1
  Version table:
     22.01+dfsg-4 500
        500 https://deb.debian.org/debian unstable/main amd64 Packages
     22.01+dfsg-2~bpo11+1 550
        550 https://deb.debian.org/debian bullseye-backports/main amd64 Packages
```

### Ejercicio 4 repos

> Añadir la arquitectura i386

```shell
sudo dpkg --add-architecture i386
sudo apt update
```

> Listar arquitecturas no nativas

```shell
vagrant@paqueteria:~$ dpkg --print-foreign-architectures
i386
```

> Eliminar la arquitectura i386

```shell
sudo dpkg --remove-architecture i386
sudo apt update
```

### Ejercicio 5 repos

> Mostrar todas las versiones disponibles de un paquete

Existen varias maneras que iré mostrando.

Con `apt policy`:

```shell
vagrant@paqueteria:~$ apt policy htop
htop:
  Installed: (none)
  Candidate: 3.0.5-7
  Version table:
     3.2.1-1 500
        500 https://deb.debian.org/debian unstable/main amd64 Packages
     3.0.5-7 600
        600 https://deb.debian.org/debian bullseye/main amd64 Packages
```

Con `apt-show-versions`:

```shell
vagrant@paqueteria:~$ apt-show-versions htop
htop:amd64 not installed
```

Esta manera tiene una gran desventaja y es como vemos, sólo muestra las versiones de lo que está instalado. Si lo instalamos, entonces sí veríamos la versión:

```shell
vagrant@paqueteria:~$ apt-show-versions htop
htop:amd64/bullseye 3.0.5-7 uptodate
```

Pero aún así, no vemos las demás versiones que podamos tener disponibles a no ser que las instalemos, así que esta manera me parece un poco incompleta.

Con `apt-cache madison`:

```shell
vagrant@paqueteria:~$ apt-cache madison htop
      htop |    3.2.1-1 | https://deb.debian.org/debian unstable/main amd64 Packages
      htop |    3.0.5-7 | https://deb.debian.org/debian bullseye/main amd64 Packages
      htop |    3.0.5-7 | https://deb.debian.org/debian bullseye/main Sources
      htop |    3.2.1-1 | https://deb.debian.org/debian unstable/main Sources
```

Con `apt-cache showpkg`:

```shell
vagrant@paqueteria:~$ apt-cache showpkg htop
Package: htop
Versions:
3.2.1-1 (/var/lib/apt/lists/deb.debian.org_debian_dists_unstable_main_binary-amd64_Packages)
 Description Language:
                 File: /var/lib/apt/lists/deb.debian.org_debian_dists_bullseye_main_binary-amd64_Packages
                  MD5: 8eb5aa19b3c92a975dc78e2165f6688d
 Description Language: en
                 File: /var/lib/apt/lists/deb.debian.org_debian_dists_bullseye_main_i18n_Translation-en
                  MD5: 8eb5aa19b3c92a975dc78e2165f6688d
 Description Language:
                 File: /var/lib/apt/lists/deb.debian.org_debian_dists_unstable_main_binary-amd64_Packages
                  MD5: 8eb5aa19b3c92a975dc78e2165f6688d

3.0.5-7 (/var/lib/apt/lists/deb.debian.org_debian_dists_bullseye_main_binary-amd64_Packages)
 Description Language:
                 File: /var/lib/apt/lists/deb.debian.org_debian_dists_bullseye_main_binary-amd64_Packages
                  MD5: 8eb5aa19b3c92a975dc78e2165f6688d
 Description Language: en
                 File: /var/lib/apt/lists/deb.debian.org_debian_dists_bullseye_main_i18n_Translation-en
                  MD5: 8eb5aa19b3c92a975dc78e2165f6688d
 Description Language:
                 File: /var/lib/apt/lists/deb.debian.org_debian_dists_unstable_main_binary-amd64_Packages
                  MD5: 8eb5aa19b3c92a975dc78e2165f6688d


Reverse Depends:
  education-common,htop
  progress-linux-base-system,htop
  live-task-extra,htop
  hollywood,htop
  freedombox,htop
  education-common,htop
  bfh-base-system,htop
  freedombox,htop
  progress-linux-base-system,htop
  live-task-extra,htop
  hollywood,htop
  freedombox,htop
Dependencies:
3.2.1-1 - libc6 (2 2.33) libncursesw6 (2 6) libnl-3-200 (2 3.2.7) libnl-genl-3-200 (2 3.2.7) libtinfo6 (2 6) lm-sensors (0 (null)) lsof (0 (null)) strace (0 (null))
3.0.5-7 - libc6 (2 2.29) libncursesw6 (2 6) libnl-3-200 (2 3.2.7) libnl-genl-3-200 (2 3.2.7) libtinfo6 (2 6) lm-sensors (0 (null)) lsof (0 (null)) strace (0 (null))
Provides:
3.2.1-1 -
3.0.5-7 -
Reverse Provides:
```

### Ejercicio 6 repos

> Comandos para descargar un paquete de stable

```shell
sudo apt install -t stable <paquete>
sudo apt install <paquete>/stable
```

### Ejercicio 7 repos

> Comandos para descargar un paquete de bullseye-backports

```shell
sudo apt install -t bullseye-backports <paquete>
sudo apt install <paquete>/bullseye-backports
```

### Ejercicio 8 repos

> Comandos para descargar un paquete de sid

```shell
sudo apt install -t unstable <paquete>
sudo apt install <paquete>/unstable
```

### Ejercicio 9 repos

> Comando para descargar un paquete de arquitectura i386

```shell
sudo apt install <paquete>:i386
```

## Trabajo con directorios

### /var/lib/apt/lists/

Con `man apt-get` nos encontramos lo siguiente:

```shell
       /var/lib/apt/lists/
           Storage area for state information for each package resource specified in sources.list(5) Configuration Item: Dir::State::Lists.
```

Es decir, en este directorio se almacena información sobre el estado de los recursos mediante los cuales obtenemos paquetes que tengamos definidos en nuestro `sources.list` (repositorios).

Al hacer `sudo apt update` este directorio se llena de datos:

```shell
vagrant@paqueteria:~$ ls -la /var/lib/apt/lists/
total 128528
drwxr-xr-x 4 root root     4096 Oct 29 15:43 .
drwxr-xr-x 5 root root     4096 Sep 12 05:17 ..
drwxr-xr-x 2 _apt root     4096 Oct 29 15:43 auxfiles
-rw-r--r-- 1 root root    48958 Oct 29 14:17 deb.debian.org_debian_dists_bullseye-backports_InRelease
-rw-r--r-- 1 root root  2102914 Oct 29 13:59 deb.debian.org_debian_dists_bullseye-backports_main_binary-amd64_Packages
-rw-r--r-- 1 root root  1641867 Oct 27 08:01 deb.debian.org_debian_dists_bullseye-backports_main_i18n_Translation-en
-rw-r--r-- 1 root root  2423808 Oct 29 08:01 deb.debian.org_debian_dists_bullseye-backports_main_source_Sources
-rw-r--r-- 1 root root    44066 Oct 29 14:17 deb.debian.org_debian_dists_bullseye-updates_InRelease
-rw-r--r-- 1 root root    56940 Oct 21 20:12 deb.debian.org_debian_dists_bullseye-updates_main_binary-amd64_Packages
-rw-r--r-- 1 root root    45666 Oct 21 20:12 deb.debian.org_debian_dists_bullseye-updates_main_i18n_Translation-en
-rw-r--r-- 1 root root    20515 Oct 21 20:12 deb.debian.org_debian_dists_bullseye-updates_main_source_Sources
-rw-r--r-- 1 root root   115947 Sep 10 10:27 deb.debian.org_debian_dists_bullseye_InRelease
-rw-r--r-- 1 root root 45528711 Sep 10 09:45 deb.debian.org_debian_dists_bullseye_main_binary-amd64_Packages
-rw-r--r-- 1 root root 30246167 Sep 10 09:45 deb.debian.org_debian_dists_bullseye_main_i18n_Translation-en
-rw-r--r-- 1 root root 44649916 Sep 10 09:45 deb.debian.org_debian_dists_bullseye_main_source_Sources
-rw-r----- 1 root root        0 Oct 29 15:43 lock
drwx------ 2 _apt root     4096 Oct 29 15:46 partial
-rw-r--r-- 1 root root    48374 Oct 29 15:02 security.debian.org_debian-security_dists_bullseye-security_InRelease
-rw-r--r-- 1 root root  1274634 Oct 27 20:43 security.debian.org_debian-security_dists_bullseye-security_main_binary-amd64_Packages
-rw-r--r-- 1 root root   909862 Oct 23 18:35 security.debian.org_debian-security_dists_bullseye-security_main_i18n_Translation-en
-rw-r--r-- 1 root root  2399825 Oct 27 20:43 security.debian.org_debian-security_dists_bullseye-security_main_source_Sources
```

Encontramos tanto ficheros con información sobre paquetes como ficheros de firmas gpg y hashes SHA256, entre otros.

### /var/lib/dpkg/available

Contiene una lista con información sobre los paquetes disponibles en sistemas basados en dpkg.

Ejemplo de contenido:

![available](https://i.imgur.com/z10Zx0P.png)

### /var/lib/dpkg/status

Es una base de datos local usada por `apt-cache` para obtener información rápidamente y sin Internet sobre los paquetes instalados.

Ejemplo de contenido:

![status](https://i.imgur.com/l7LFOzr.png)

### /var/cache/apt/archives/

Es el directorio usado para almacenar los paquetes .deb descargados. Tiene un subdirectorio `partial` que funciona como almacenamiento para los paquetes en tránsito.
