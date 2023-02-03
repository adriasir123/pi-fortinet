# Montaje NFS mediante systemd

## 1. Enunciado

En un servidor Debian 11 del cloud conectar un volumen de 2GB. Luego:

- Hacer un montaje con una unidad de systemd del volumen
- Compartir el directorio con un servidor nfs

En un cliente Debian 11 del cloud:

- Con un cliente nfs y una unidad de systemd montar el directorio compartido

## 2. Pasos previos

Muestro que tengo el volumen correctamente enlazado:

![volumen2g](https://i.imgur.com/j76lbA3.png)

Lo formateo:

```shell
sudo mkfs -t ext4 /dev/vdb
```

## 3. En el servidor

### 3.1 Unidad de montaje

Creo la unidad de montaje:

```shell
sudo nano /etc/systemd/system/mnt.mount
```

```shell
[Unit]
Description=vdb 2G volume

[Mount]
What=/dev/vdb
Where=/mnt
Type=ext4
Options=defaults

[Install]
WantedBy=multi-user.target
```

La inicio y la habilito en el arranque:

```shell
sudo systemctl start mnt.mount
sudo systemctl enable mnt.mount
```

### 3.2 NFS server

Instalo el paquete:

```shell
sudo apt update
sudo apt install nfs-kernel-server
```

Defino el directorio compartido en el siguiente fichero:

```shell
sudo nano /etc/exports
```

```shell
/mnt 10.0.0.0/24(rw,sync,no_subtree_check,no_root_squash)
```

Reinicio nfs y actualizo la lista de directorios compartidos:

```shell
sudo systemctl restart nfs-kernel-server
sudo exportfs
```

### 3.3 Pruebas

Unidad funcionando:

```shell
debian@systemd-nfs:~$ sudo systemctl status mnt.mount
● mnt.mount - vdb 2G volume
     Loaded: loaded (/proc/self/mountinfo; enabled; vendor preset: enabled)
     Active: active (mounted) since Fri 2023-02-03 16:18:11 UTC; 45min ago
      Where: /mnt
       What: /dev/vdb
      Tasks: 0 (limit: 527)
     Memory: 28.0K
        CPU: 5ms
     CGroup: /system.slice/mnt.mount

Feb 03 16:18:11 systemd-nfs systemd[1]: Mounting vdb 2G volume...
Feb 03 16:18:11 systemd-nfs systemd[1]: Mounted vdb 2G volume.
```

Volumen montado:

![mntmontado](https://i.imgur.com/bS0RYPD.png)

## 4. En el cliente

### 4.1 NFS client

```shell
sudo apt update
sudo apt install nfs-common
```

### 4.2 Unidad de montaje

Creo la unidad de montaje:

```shell
sudo nano /etc/systemd/system/mnt.mount
```

```shell
[Unit]
Description=NFS mount
After=network-online.target
Wants=network-online.target

[Mount]
What=10.0.0.186:/mnt
Where=/mnt
Options=auto
Type=nfs
TimeoutSec=60

[Install]
WantedBy=remote-fs.target
```

La inicio y la habilito en el arranque:

```shell
sudo systemctl start mnt.mount
sudo systemctl enable mnt.mount
```

### 4.3 Pruebas

Unidad funcionando:

```shell
debian@systemd-nfs-client:~$ sudo systemctl status mnt.mount
● mnt.mount - NFS mount
     Loaded: loaded (/proc/self/mountinfo; enabled; vendor preset: enabled)
     Active: active (mounted) since Fri 2023-02-03 16:59:38 UTC; 12min ago
      Where: /mnt
       What: 10.0.0.186:/mnt
      Tasks: 0 (limit: 527)
     Memory: 76.0K
        CPU: 6ms
     CGroup: /system.slice/mnt.mount

Feb 03 16:59:37 systemd-nfs-client systemd[1]: Mounting NFS mount...
Feb 03 16:59:38 systemd-nfs-client systemd[1]: Mounted NFS mount.
```

Directorio remoto montado:

![mntmontadonfs](https://i.imgur.com/bOjHpWQ.png)

Contenido:

```shell
debian@systemd-nfs-client:~$ ls -la /mnt/
total 24
drwxr-xr-x  3 root root  4096 Feb  3 16:17 .
drwxr-xr-x 18 root root  4096 Feb  3 16:43 ..
drwx------  2 root root 16384 Feb  3 14:33 lost+found
-rw-r--r--  1 root root     0 Feb  3 16:17 test
```
