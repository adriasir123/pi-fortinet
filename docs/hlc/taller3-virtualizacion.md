# Taller 3: Creación de máquinas virtuales con virt-manager

La teoría para completar este taller se encuentra en el [capítulo 4 del curso de virtualización](https://github.com/josedom24/curso_virtualizacion_linux).

## Entrega

### Parte 1

> Mostrar el acceso a `debian_adrian_j`

![accesovmanterior](https://i.imgur.com/eMpMCGc.png)

### Parte 2

> Mostrar el acceso a `debian_adrian_j` por ssh

![porssh](https://i.imgur.com/WXYgSQm.png)

### Parte 3

> Mostrar el XML de `debian_adrian_j`

![xmlmaquina](https://i.imgur.com/qN8nOiD.png)

### Parte 4

> Sobre `debian_adrian_j`, mostrar:
>
> - RAM
> - CPUs
> - Disco
> - Interfaz de red

![memoria](https://i.imgur.com/rs80Vdw.png)

![cpus](https://i.imgur.com/tbQENGv.png)

![discos](https://i.imgur.com/VAnuC7u.png)

![interfazred](https://i.imgur.com/fJR8q8J.png)

### Parte 5

> Muestro un ping al exterior en la VM Windows

![windowsping](https://i.imgur.com/UHWLRwa.png)

### Parte 6

> Muestro la vista general XML, el disco y la NIC VirtIO de la VM Windows

![xmlwin10](https://i.imgur.com/DMyigNO.png)

![discovirtio](https://i.imgur.com/guZGv22.png)

![nicvirtio](https://i.imgur.com/Bkyjnrn.png)

## Desarrollo

### Ejercicio 1

> Instalar `virt-manager`

```shell
sudo apt install virt-manager
```

### Ejercicio 2

> Acceder a `debian_adrian_j` del taller 2

![accesovmanterior](https://i.imgur.com/eMpMCGc.png)

### Ejercicio 3

> Instalar un servidor ssh en `debian_adrian_j` y acceder

```shell
sudo apt install openssh-server
```

```shell
atlas@olympus:~$ ssh debian@192.168.122.47
The authenticity of host '192.168.122.47 (192.168.122.47)' can't be established.
ECDSA key fingerprint is SHA256:u1XoocfVaaUcYJ85av22BbCiS7hZ73mFSGHNUbcev0M.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.122.47' (ECDSA) to the list of known hosts.
debian@192.168.122.47's password:
Linux debian 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Oct 10 14:04:52 2022
debian@debian:~$
```

### Ejercicio 4

> Mostrar el XML de `debian_adrian_j`

![xmlmaquina](https://i.imgur.com/qN8nOiD.png)

### Ejercicio 5

> Sobre `debian_adrian_j`, mostrar:
>
> - RAM
> - CPUs
> - Disco
> - Interfaz de red

![memoria](https://i.imgur.com/rs80Vdw.png)

![cpus](https://i.imgur.com/tbQENGv.png)

![discos](https://i.imgur.com/VAnuC7u.png)

![interfazred](https://i.imgur.com/fJR8q8J.png)

### Ejercicio 6

> Crear una VM con Windows, disco y NIC VirtIO

Muestro que tengo las isos necesarias:

![isos](https://i.imgur.com/I55EZGd.png)

Creo la máquina:

![creowindows1](https://i.imgur.com/hBRXb87.png)

![creowindows2](https://i.imgur.com/RQ0ALjp.png)

![creowindows3](https://i.imgur.com/3V7UzCv.png)

![creowindows4](https://i.imgur.com/oxxRwYp.png)

En el paso final, personalizo la configuración de la máquina antes de instalar:

![creowindows5](https://i.imgur.com/BcvFnry.png)

Cambio el disco a VirtIO:

![creowindows6](https://i.imgur.com/596e8Ys.png)

Cambio la NIC a VirtIO:

![creowindows7](https://i.imgur.com/jPqVyTf.png)

Añado un CDROM con la ISO de drivers VirtIO:

![creowindows8](https://i.imgur.com/LURk1Qs.png)

Cambio el orden de arranque y lanzo la instalación:

![creowindows9](https://i.imgur.com/l7dIJKd.png)

Cargo los drivers para que se reconozca el disco:

![creowindows10](https://i.imgur.com/S3buYO2.png)

![creowindows11](https://i.imgur.com/qnQjVoX.png)

![creowindows12](https://i.imgur.com/F9lhD0P.png)

![creowindows13](https://i.imgur.com/v0mLoJu.png)

Ya podemos ver el disco y terminar la instalación:

![creowindows14](https://i.imgur.com/JfPaBR5.png)

Instalo los drivers VirtIO para que funcione la NIC:

![creowindows15](https://i.imgur.com/lIBQcvm.png)

![creowindows16](https://i.imgur.com/65Su1IL.png)

![creowindows17](https://i.imgur.com/iYikgjQ.png)

Hago un ping a Internet para probar que funciona:

![creowindows18](https://i.imgur.com/UHWLRwa.png)
