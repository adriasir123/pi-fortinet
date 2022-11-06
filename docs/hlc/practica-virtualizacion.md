# Práctica: Virtualización en Linux

## Entrega

### Parte 1

> URL del repositorio GitHub

<https://github.com/adriasir123/script-virtualizacion>

### Parte 2

> Indicar los pasos para crear la imagen base

En el [siguiente apartado](#creación-de-imagen-base) se explica.

### Parte 3

> Entregar la clave privada y un enlace para descargar la imagen base

Todo se encuentra en el repositorio.

### Parte 4

> Ejecutar el script y cuando se pause mostrar captura accediendo al servidor web de maquina1

![primeraccesoweb](https://i.imgur.com/Zk4QsbM.png)

### Parte 5

> Mostrar captura accediendo al servidor web con la IP pública al terminar el script

![accesowebpublico](https://i.imgur.com/kWEaKA2.png)

### Parte 6

> Mostrar captura del disco montado en /var/www/html

![vdbmontado](https://i.imgur.com/Dt4L5jf.png)

> Mostrar captura de los 2G de RAM que tiene la máquina

![2gram](https://i.imgur.com/LCz8ocv.png)

> Mostrar captura accediendo al contenedor

![accesocontenedor](https://i.imgur.com/0bwTXsT.png)

> Mostrar captura de la snapshot creada

![snapshot](https://i.imgur.com/oNxoXZu.png)

## Creación de imagen base

### Redes

Creo el fichero:

```shell
nano red-script-1.xml
```

Con el contenido:

```shell
<network>
  <name>red-script-1</name>
  <bridge name='virbr-script-1'/>
  <forward/>
  <ip address='192.168.123.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.123.2' end='192.168.123.254'/>
    </dhcp>
  </ip>
</network>
```

Creo la red `red-script-1`:

```shell
virsh -c qemu:///system net-define red-script-1.xml
```

Creo el fichero:

```shell
nano red-script-2.xml
```

Con el contenido:

```shell
<network>
  <name>red-script-2</name>
  <bridge name='virbr-script-2'/>
  <forward/>
  <ip address='192.168.124.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.124.2' end='192.168.124.254'/>
    </dhcp>
  </ip>
</network>
```

Creo la red `red-script-2`:

```shell
virsh -c qemu:///system net-define red-script-2.xml
```

Las inicio:

```shell
virsh -c qemu:///system net-start red-script-1
virsh -c qemu:///system net-start red-script-2
```

### VM

```shell
virt-install --connect qemu:///system \
             --virt-type kvm \
             --name bullseye-base \
             --cdrom ~/isos/debian-11.5.0-amd64-netinst.iso \
             --os-variant debian10 \
             --network network=red-script-1 \
             --network network=red-script-2 \
             --disk size=3 \
             --memory 2048 \
             --vcpus 2
```

### Sistema de ficheros

![xfs](https://i.imgur.com/aDIhUrl.png)

### Interfaces

Añado las siguientes líneas a `/etc/network/interfaces`:

```shell
nano /etc/network/interfaces
```

```shell
# The secondary network interface
allow-hotplug enp2s0
iface enp2s0 inet dhcp
```

### Usuario

> Hacer que el usuario `debian` con contraseña debian utilice sudo sin contraseña

Sigo estas órdenes:

```shell
su -
apt install sudo
usermod -aG sudo debian
visudo
```

Al final del fichero añado:

```shell
debian ALL=(ALL) NOPASSWD:ALL
```

### SSH

> Crear un keypair ssh de formato ecdsa sin passphrase

```shell
ssh-keygen -t ecdsa
```

> Insertar la clave pública al usuario debian

```shell
ssh-copy-id -i script.pub debian@192.168.123.49
```

> Probar el acceso

```shell
ssh -i script debian@192.168.123.49
```

### Reinicio

```shell
reboot
```

### Reducción imagen base

> Utilizar `virt-sparsify` para reducir el tamaño de la imagen

Tengo que tener la máquina parada:

```shell
virsh -c qemu:///system destroy bullseye-base
```

Muevo el fichero de disco al directorio necesario:

```shell
sudo mv /var/lib/libvirt/images/bullseye-base.qcow2  ~/script-virtualizacion
```

Modifico propietarios:

```shell
sudo chown atlas:atlas bullseye-base.qcow2
```

Reduzco la imagen:

```shell
virt-sparsify bullseye-base.qcow2 bullseye-base-sparse.qcow2
```

Ya puedo borrar la máquina:

```shell
virsh -c qemu:///system undefine bullseye-base
```

Y la imagen original:

```shell
rm bullseye-base.qcow2
```

## Script

Se encuentra en [este repositorio](https://github.com/adriasir123/script-virtualizacion), y allí encontrarás también el enlace para descargar la imagen base sparse.

[Aquí](https://gist.github.com/adriasir123/dc9d3330cf0b95600180f7c5148396bf) puedes ver el output completo del script cuando no hay nada creado.

[Aquí](https://gist.github.com/adriasir123/ba84ee8bae04e280229cd45d584190b9) puedes ver el output completo del script cuando todo está creado.
