# Compilación de programas en C

## Escenario

```shell
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 1
    libvirt.memory = 512
  end

  config.vm.define :compilacion do |compilacion|
    compilacion.vm.box = "debian/bullseye64"
    compilacion.vm.hostname = "compilacion"
  end

end
```

## Apache 2.4.54

### Dependencias

```shell
sudo apt update
sudo apt build-dep apache2 
```

### Código fuente

Cambio a un directorio apropiado:

```shell
cd /usr/local/src
```

Descargo:

```shell
sudo wget https://dlcdn.apache.org/httpd/httpd-2.4.54.tar.gz
```

Extraigo:

```shell
sudo gzip -d httpd-2.4.54.tar.gz
sudo tar xvf httpd-2.4.54.tar
```

Entro al directorio generado:

```shell
cd httpd-2.4.54
```

### Generación de Makefile

Creo el directorio que necesitaremos:

```shell
sudo mkdir /usr/local/apache2.4.54
```

Genero:

```shell
sudo ./configure --prefix=/usr/local/apache2.4.54
```

### Compilación

```shell
sudo make
```

### Instalación

```shell
sudo make install
```

### Funcionamiento

Iniciamos el servidor:

```shell
sudo /usr/local/apache2.4.54/bin/apachectl -k start
```

Mostramos la web por defecto accediendo a la IP de la VM:

![apachefuncionando](https://i.imgur.com/PjSYclU.png)

El html que vemos se encuentra en `/usr/local/apache2.4.54/htdocs/index.html`

Podemos parar el servidor así:

```shell
sudo /usr/local/apache2.4.54/bin/apachectl -k stop
```

Al recargar la página veríamos que no funciona:

![apacheparado](https://i.imgur.com/de4yR40.png)

### Desinstalación

Simplemente tendríamos que borrar el directorio donde hicimos la instalación, ya que este código fuente no nos provee de la función `make uninstall`:

```shell
sudo rm -r /usr/local/apache2.4.54
```

Después de un reinicio de la VM nuestro Apache dejaría de funcionar.

## Nala

### Añadir Testing

Dejo `/etc/apt/sources.list` de la siguiente manera:

```shell
deb https://deb.debian.org/debian bullseye main
deb https://deb.debian.org/debian testing main
deb-src https://deb.debian.org/debian bullseye main
deb https://security.debian.org/debian-security bullseye-security main
deb-src https://security.debian.org/debian-security bullseye-security main
deb https://deb.debian.org/debian bullseye-updates main
deb-src https://deb.debian.org/debian bullseye-updates main
deb https://deb.debian.org/debian bullseye-backports main
deb-src https://deb.debian.org/debian bullseye-backports main
```

### Requerimientos

```shell
sudo apt update
sudo apt install git python3-apt pandoc build-essential python3-pip fish
```

### Clonar main

```shell
git clone https://gitlab.com/volian/nala.git
cd nala
```

### Instalar

```shell
sudo make install
```

Comprobamos que funciona:

```shell
vagrant@compilacion:~$ nala --version
nala 0.11.1
```

### Desinstalar

```shell
sudo make uninstall
```

### VS apt

- Descargas en paralelo
- `nala fetch` elegirá los 3 mirrors más rápidos con respecto a latencia (ms) y nos los guardará en un fichero
- `nala history` guarda las transacciones
- Mejora visualmente

[Aquí](https://ostechnix.com/nala-commandline-frontend-for-apt/) podemos ver muchas funciones más que puede hacer Nala.

### Opiniones

Creo que Nala aún está en un estado un poco verde, podría tener muchas más funciones, pero actualmente se encuentra oficialmente añadido en testing y en unstable.

Esto quiere decir que es probable que en la siguiente release stable de Debian (Bookworm) contemos con Nala como paquete oficial.

Nala es un frontend de apt como lo son Aptitude y Synaptic, inspirado en dnf, y tan sólo por lo vistoso que es y la rapidez que ofrece le veo mucho futuro si se sigue con el desarrollo.
