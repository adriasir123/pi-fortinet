# Práctica: app web PHP en Docker

## Info previa

Crearemos varias versiones de una imagen de [BookMedik](https://github.com/evilnapsis/bookmedik) que implantaremos.

## Tarea 1: Creación de una imagen docker con una aplicación web desde una imagen base

Muestro el directorio donde construiré la imagen:

```console
atlas@olympus:~/docker/bookmedikv1build$ ls -la
total 8
drwxr-xr-x  2 atlas atlas 4096 Jun 18 15:47 .
drwxr-xr-x 19 atlas atlas 4096 Jun 18 15:47 ..
```

Me bajo aquí el código de BookMedik:

```console
atlas@olympus:~/docker/bookmedikv1build$ git clone https://github.com/evilnapsis/bookmedik.git
Cloning into 'bookmedik'...
remote: Enumerating objects: 856, done.
remote: Total 856 (delta 0), reused 0 (delta 0), pack-reused 856
Receiving objects: 100% (856/856), 1.90 MiB | 2.25 MiB/s, done.
Resolving deltas: 100% (372/372), done.
atlas@olympus:~/docker/bookmedikv1build$ ls -la
total 12
drwxr-xr-x  3 atlas atlas 4096 Jun 18 16:46 .
drwxr-xr-x 19 atlas atlas 4096 Jun 18 15:47 ..
drwxr-xr-x  7 atlas atlas 4096 Jun 18 16:46 bookmedik
```

Elimino las siguientes líneas de `schema.sql`:

```console
create database bookmedik;
use bookmedik;
```

Modifico `core/controller/Database.php` de la siguiente manera:

```console
<?php
class Database {
	public static $db;
	public static $con;
	function Database(){
	        $this->user=getenv('BOOKMEDIK_DB_USER');$this->pass=getenv('BOOKMEDIK_DB_PASSWORD');$this->host=getenv('BOOKMEDIK_DB_HOST');$this->ddbb=getenv('BOOKMEDIK_DB_NAME');
	}

	function connect(){
	        $con = new mysqli($this->host,$this->user,$this->pass,$this->ddbb);
	        $con->query("set sql_mode=''");
	        return $con;
	}

	public static function getCon(){
	        if(self::$con==null && self::$db==null){
		                self::$db = new Database();
		                self::$con = self::$db->connect();
	        }
	        return self::$con;
	}

}
?>
```

Creo aquí `script.sh`:

```console
#! /bin/sh

sleep 10

mysql -u $BOOKMEDIK_DB_USER --password=$BOOKMEDIK_DB_PASSWORD -h $BOOKMEDIK_DB_HOST $BOOKMEDIK_DB_NAME < /var/www/html/schema.sql

/usr/sbin/apache2ctl -D FOREGROUND
```

Crearé el siguiente `Dockerfile`:

```console
FROM debian
MAINTAINER Adrián Jaramillo Rodríguez "adristudy@gmail.com"
RUN apt-get update && apt-get install -y apache2 libapache2-mod-php php php-mysql mariadb-client && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN rm /var/www/html/index.html
ADD bookmedik /var/www/html/
ADD script.sh /opt/
RUN chmod +x /opt/script.sh
EXPOSE 80
ENTRYPOINT ["/opt/script.sh"]
```

Genero la imagen:

```console
docker build -t adrianjaramillo/bookmedik:v1 .
```

### Entregas tarea 1

[Repositorio para la generación de la imagen v1](https://github.com/adriasir123/bookmedik-v1-build)

Imagen generada localmente:

![bookmedikv1](https://i.imgur.com/84owJ12.png)

## Tarea 2: Despliegue en el entorno de desarrollo

Muestro el directorio donde lanzaré el escenario:

```console
atlas@olympus:~/docker/bookmedikescenario$ ls -la
total 8
drwxr-xr-x  2 atlas atlas 4096 Jun 18 18:08 .
drwxr-xr-x 20 atlas atlas 4096 Jun 18 18:08 ..
```

Crearé el siguiente `docker-compose.yml`:

```console
version: '3.1'
services:
  bookmedik:
    container_name: bookmedik
    image: adrianjaramillo/bookmedik:v1
    restart: always
    environment:
      BOOKMEDIK_DB_USER: bookmedik_user
      BOOKMEDIK_DB_PASSWORD: bookmedik_pass
      BOOKMEDIK_DB_NAME: bookmedik
      BOOKMEDIK_DB_HOST: mariadb
    ports:
      - 80:80
    depends_on:
      - mariadb
    volumes:
      - bookmedik_logs:/var/log/apache2
  mariadb:
    container_name: mariadb
    image: mariadb
    restart: always
    environment:
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: bookmedik_user
      MARIADB_PASSWORD: bookmedik_pass
      MARIADB_ROOT_PASSWORD: root
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    bookmedik_logs:
    mariadb_data:
```

Lanzo los contenedores:

```console
atlas@olympus:~/docker/bookmedikescenario$ docker-compose up -d
Creating network "bookmedikescenario_default" with the default driver
Creating volume "bookmedikescenario_bookmedik_logs" with default driver
Creating volume "bookmedikescenario_mariadb_data" with default driver
Creating mariadb ... done
Creating bookmedik ... done
```

### Entregas tarea 2

[Repositorio para la creación del escenario](https://github.com/adriasir123/bookmedik-escenario)

Escenario funcionando:

![escenariofuncionando](https://i.imgur.com/fn5XWjT.png)

BookMedik funcionando:

![bookmedikfuncionando](https://i.imgur.com/tCZ7P5Q.png)

## Tarea 3: Creación de una imagen docker con una aplicación web desde una imagen PHP

Copio el directorio de construcción de la imagen anterior, ya que esta versión de la imagen será *muy parecida* a la anterior:

```console
cp -r bookmedikv1build bookmedikv2build
```

Limpio el directorio `.git` antiguo:

```console
sudo rm -r .git
```

Modifico el `Dockerfile`:

```console
FROM php:7.4-apache
MAINTAINER Adrián Jaramillo Rodríguez "adristudy@gmail.com"
RUN apt-get update && apt-get install -y mariadb-client && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-install mysqli pdo pdo_mysql && docker-php-ext-enable pdo_mysql
ADD bookmedik /var/www/html/
ADD script.sh /opt/
RUN chmod +x /opt/script.sh
EXPOSE 80
ENTRYPOINT ["/opt/script.sh"]
```

Genero la imagen:

```console
docker build -t adrianjaramillo/bookmedik:v2 .
```

Modifico el `docker-compose.yml` para que use la nueva imagen:

```console
version: '3.1'
services:
  bookmedik:
    container_name: bookmedik
    image: adrianjaramillo/bookmedik:v2
    restart: always
    environment:
      BOOKMEDIK_DB_USER: bookmedik_user
      BOOKMEDIK_DB_PASSWORD: bookmedik_pass
      BOOKMEDIK_DB_NAME: bookmedik
      BOOKMEDIK_DB_HOST: mariadb
    ports:
      - 80:80
    depends_on:
      - mariadb
    volumes:
      - bookmedik_logs:/var/log/apache2
  mariadb:
    container_name: mariadb
    image: mariadb
    restart: always
    environment:
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: bookmedik_user
      MARIADB_PASSWORD: bookmedik_pass
      MARIADB_ROOT_PASSWORD: root
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    bookmedik_logs:
    mariadb_data:
```

Lanzo los contenedores:

```console
atlas@olympus:~/docker/bookmedikescenario$ docker-compose up -d
Creating network "bookmedikescenario_default" with the default driver
Creating volume "bookmedikescenario_bookmedik_logs" with default driver
Creating volume "bookmedikescenario_mariadb_data" with default driver
Creating mariadb ... done
Creating bookmedik ... done
```

### Entregas tarea 3

[Repositorio para la generación de la imagen v2](https://github.com/adriasir123/bookmedik-v2-build)

Imagen generada localmente:

![bookmedikv2](https://i.imgur.com/uUVqGSU.png)

Escenario funcionando:

![escenariofuncionandov2](https://i.imgur.com/CIooWUF.png)

BookMedik funcionando:

![bookmedikfuncionandov2](https://i.imgur.com/9oWtXCg.png)

## Tarea 5: Puesta en producción de nuestra aplicación



















