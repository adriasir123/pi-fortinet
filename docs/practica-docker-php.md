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

### Renovación de certificado

Voy a documentar una situación que se me ha dado personalmente al intentar poner en producción la imagen de Docker, que no se pide explícitamente que se documente durante la práctica, pero que me parece de vital importancia el saber hacerlo porque es algo que nos puede suceder a todos durante el curso....

**¡Mi certificado Let's Encrypt ha expirado, necesita renovación!**

En mi caso era un wilcard para `*.adrianjaramillo.tk`, por lo que para renovarlo básicamente tenemos que repetir el proceso que hicimos para su primera expedición.

Por ahora, muestro que tenemos un error de certificado expirado:

![certexpirado](https://i.imgur.com/nsb3uVf.png)

Inicio la petición del certificado:

```console
blackmamba@kampe:~$ sudo certbot certonly --manual --preferred-challenges dns -d *.adrianjaramillo.tk
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Cert is due for renewal, auto-renewing...
Renewing an existing certificate for *.adrianjaramillo.tk
Performing the following challenges:
dns-01 challenge for adrianjaramillo.tk

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.adrianjaramillo.tk with the following value:

K2G9aLEv8hrXEthRTua27QfKkXyI8iWCAkh8NntOcmQ

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

Cerbot se ha dado cuenta de que el certificado ha expirado y necesita renovación, es decir, sabe que no está pidiendo un certificado nuevo.

Añado el siguiente registro a mi DNS:

![dns01challenge](https://i.imgur.com/kkCSlfs.png)

Tras comprobar con un `dig` que el registro se ha publicado, finalizo la renovación:

```console
blackmamba@kampe:~$ sudo certbot certonly --manual --preferred-challenges dns -d *.adrianjaramillo.tk
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Cert is due for renewal, auto-renewing...
Renewing an existing certificate for *.adrianjaramillo.tk
Performing the following challenges:
dns-01 challenge for adrianjaramillo.tk

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.adrianjaramillo.tk with the following value:

K2G9aLEv8hrXEthRTua27QfKkXyI8iWCAkh8NntOcmQ

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/adrianjaramillo.tk/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/adrianjaramillo.tk/privkey.pem
   Your certificate will expire on 2022-09-17. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Verifico que el certificado está renovado y vuelve a ser válido:

```console
blackmamba@kampe:~$ sudo certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: adrianjaramillo.tk
    Serial Number: 4ebd40230aa2fa7404df1db4bb401513314
    Key Type: RSA
    Domains: *.adrianjaramillo.tk
    Expiry Date: 2022-09-17 12:04:09+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/adrianjaramillo.tk/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/adrianjaramillo.tk/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

Es necesario reiniciar Nginx ya que si no se hace, no ofrecerá el certificado renovado:

```console
sudo systemctl restart nginx
```

Muestro que ya tengo acceso a mi web sin problemas:

![certrenovado](https://i.imgur.com/4dABhAQ.png)

### Publicación de imagen

Subo la v2 a Docker Hub:

```console
atlas@olympus:~$ docker push adrianjaramillo/bookmedik:v2
The push refers to repository [docker.io/adrianjaramillo/bookmedik]
1010d1bbc144: Pushed
771eec85b637: Pushed
6f115feedd78: Pushed
ce64b16b53a7: Pushed
1cb3eb48df9d: Pushed
7480afd8fa60: Mounted from library/wordpress
027c5bd47ef8: Mounted from library/wordpress
13b107c2dc5f: Mounted from library/wordpress
484b057405b5: Mounted from library/wordpress
a9157dcd056e: Mounted from library/wordpress
73d6f16bc909: Mounted from library/wordpress
ff48eea959aa: Mounted from library/wordpress
e71e1bcde4cb: Mounted from library/wordpress
c41e2fe9ff49: Mounted from library/wordpress
fc39acdd710d: Mounted from library/wordpress
df29ad355f78: Mounted from library/wordpress
ca59c88c69a3: Mounted from library/wordpress
ad6562704f37: Mounted from adrianjaramillo/mi_servidor_web
v2: digest: sha256:77c06993dcf3988354fcc79eb86506fbc01bf7031690a0f72b80bd2c031cb0bc size: 4081
```

### Registro DNS

![bookmedikdns](https://i.imgur.com/nue6EL7.png)

### Preparación de Docker en VPS

```console
sudo apt update
sudo apt install docker.io
sudo usermod -aG docker blackmamba
sudo apt install docker-compose
```

### Preparación de Nginx

Configuraré el par de VirtualHosts *(HTTP y HTTPS)* para BookMedik.

`bookmedik_http.conf`:

```console
server {

    listen 80;
    server_name bookmedik.adrianjaramillo.tk;

    return 301 https://$host$request_uri;

}
```

Lo activo:

```console
sudo ln -s /etc/nginx/sites-available/bookmedik_http.conf /etc/nginx/sites-enabled/
```

`bookmedik_https.conf`:

```console
server {

    listen 443 ssl;
    server_name bookmedik.adrianjaramillo.tk;

    ###################
    # RSA certificate #
    ###################

    ssl_certificate /etc/letsencrypt/live/adrianjaramillo.tk/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/adrianjaramillo.tk/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;

    #############
    # Locations #
    #############

    location / {
        proxy_pass http://localhost:8081;
        include proxy_params;
    }

}
```

Lo activo:

```console
sudo ln -s /etc/nginx/sites-available/bookmedik_https.conf /etc/nginx/sites-enabled/
```

Reinicio Nginx:

```console
sudo systemctl restart nginx
```

### Despliegue de BookMedik

Clono mi repositorio que tiene el `docker-compose.yml` que necesito:

```console
blackmamba@kampe:~/docker$ git clone git@github.com:adriasir123/bookmedik-escenario.git
Cloning into 'bookmedik-escenario'...
Warning: Permanently added the ECDSA host key for IP address '140.82.121.3' to the list of known hosts.
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 1), reused 6 (delta 1), pack-reused 0
Receiving objects: 100% (6/6), done.
Resolving deltas: 100% (1/1), done.
```

Cambio el puerto real en el que se servirá BookMedik, ya que el 80 y el 8080 los tengo ocupados en mi VPS:

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
      - 8081:80
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

Me bajo las imágenes:

```console
blackmamba@kampe:~/docker/bookmedik-escenario$ docker pull adrianjaramillo/bookmedik:v2
v2: Pulling from adrianjaramillo/bookmedik
42c077c10790: Pull complete
8934009a9160: Pull complete
5357ac116991: Pull complete
54ae63894b5a: Pull complete
772088206f85: Pull complete
3b81c5474649: Pull complete
c62a528527ae: Pull complete
344c792709d4: Pull complete
ef9638273c35: Pull complete
4ec70638e42f: Pull complete
5a0f65f5bb3b: Pull complete
55578a5ce5e8: Pull complete
f1dd85b4dfa5: Pull complete
f42db0cb760d: Pull complete
3fe0a43063d9: Pull complete
5630d2e894d6: Pull complete
1178877b058a: Pull complete
8a3b5072a55a: Pull complete
Digest: sha256:77c06993dcf3988354fcc79eb86506fbc01bf7031690a0f72b80bd2c031cb0bc
Status: Downloaded newer image for adrianjaramillo/bookmedik:v2
docker.io/adrianjaramillo/bookmedik:v2
blackmamba@kampe:~/docker/bookmedik-escenario$ docker pull mariadb
Using default tag: latest
latest: Pulling from library/mariadb
405f018f9d1d: Pull complete
7a85079b8234: Pull complete
579c7ff691b1: Pull complete
4976663b5d6d: Pull complete
169024b1fb13: Pull complete
c0ffe8ce897f: Pull complete
b583c09d23c3: Pull complete
9b9f0c08d08f: Pull complete
9cd51f984586: Pull complete
d9f506bb8aca: Pull complete
24d689f79ba4: Pull complete
Digest: sha256:88fcb7d92c7f61cd885c4d309c98461f3607aa6dbd57a2474be86e1956b36d13
Status: Downloaded newer image for mariadb:latest
docker.io/library/mariadb:latest
```

Compruebo que las tengo:

```console
blackmamba@kampe:~/docker/bookmedik-escenario$ docker images
REPOSITORY                  TAG       IMAGE ID       CREATED        SIZE
adrianjaramillo/bookmedik   v2        64331ad6b6f9   15 hours ago   496MB
mariadb                     latest    ea81af801379   12 days ago    383MB
```

Lanzo el escenario:

```console
blackmamba@kampe:~/docker/bookmedik-escenario$ docker-compose up -d
Creating network "bookmedik-escenario_default" with the default driver
Creating volume "bookmedik-escenario_bookmedik_logs" with default driver
Creating volume "bookmedik-escenario_mariadb_data" with default driver
Creating mariadb ... done
Creating bookmedik ... done
```

### Entregas tarea 5

Muestro mi imagen v2 en Docker Hub:

![dockerhubv2mia](https://i.imgur.com/XJwZ0uJ.png)

Mi configuración de Nginx ya se ha mostrado [aquí](####-preparacion-de-nginx).









