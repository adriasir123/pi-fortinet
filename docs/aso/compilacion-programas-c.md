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

Añadido a la práctica: compilar nala y evaluar sus posibilidades, decir nuestra opinión




















