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

```shell

```








## Trabajo con ficheros .deb

### Ejercicio 1

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

### Ejercicio 2

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

### Ejercicio 3

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
