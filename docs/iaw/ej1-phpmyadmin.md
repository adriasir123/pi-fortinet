# Ejercicio 1: Instalación de phpmyadmin


## ENTREGA

### Parte 1
> Captura donde se vea la base de datos creada en el punto 1

![](https://i.imgur.com/jGOUM30.png)

### Parte 2
> ¿Cómo has quitado la configuración de acceso a phpmyadmin en el punto 5?

```
sudo a2disconf phpmyadmin
sudo systemctl restart apache2
```

### Parte 3
> Captura del VirtualHost

![](https://i.imgur.com/Mzphzd1.png)

### Parte 4
> Captura accediendo a phpmyadmin con el usuario del punto 1

![](https://i.imgur.com/OdWmih8.png)

A la derecha en rojo muestro el campo donde se indica con qué usuario hemos iniciado sesión.



## REALIZACIÓN

### Paso 0
> Instalar `mariadb-server`

```
sudo apt update
sudo apt install mariadb-server
```

> Hacer que el usuario root acceda con contraseña

Por defecto el usuario root puede hacer login sin contraseña, porque internamente mariadb lo almacena con una password inválida:
```
MariaDB [(none)]> SELECT host, user, password FROM mysql.user;
+-----------+-------------+----------+
| Host      | User        | Password |
+-----------+-------------+----------+
| localhost | mariadb.sys |          |
| localhost | root        | invalid  |
| localhost | mysql       | invalid  |
+-----------+-------------+----------+
```

Este funcionamiento es intencional, para que luego nosotros manualmente cambiemos a la contraseña que queramos.

Cambio la contraseña con:
```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'vagrant';
```

Después de esto, la contraseña se almacena:
```
MariaDB [(none)]> SELECT host, user, password FROM mysql.user;
+-----------+-------------+-------------------------------------------+
| Host      | User        | Password                                  |
+-----------+-------------+-------------------------------------------+
| localhost | mariadb.sys |                                           |
| localhost | root        | *04E6E1273D1783DF7D57DC5479FE01CFFDFD0058 |
| localhost | mysql       | invalid                                   |
+-----------+-------------+-------------------------------------------+
```

Ya podríamos hacer login normalmente con:
```
mysql -u root -p
```

A partir de ahora, de cualquier manera que intentemos entrar con root (*con sudo o sin sudo por ejemplo*), nos obligará a escribir la contraseña.


### Paso 1
> Acceder a mariadb con root y contraseña

```
mysql -u root -p
```

> Crear base de datos

```
CREATE DATABASE `phpmyadmin_db`;
```

> Crear usuario con permisos sobre `phpmyadmin_db`

```
CREATE USER 'phpmyadmin_user' IDENTIFIED BY '1234';

GRANT USAGE ON *.* TO 'phpmyadmin_user'@localhost IDENTIFIED BY '1234';

GRANT ALL privileges ON `phpmyadmin_db`.* TO 'phpmyadmin_user'@localhost;

FLUSH PRIVILEGES;
```

Muestro que los cambios se han hecho:
```
MariaDB [(none)]> SHOW GRANTS FOR 'phpmyadmin_user'@localhost;   
+------------------------------------------------------------------------------------------------------------------------+
| Grants for phpmyadmin_user@localhost                                                                                   |
+------------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `phpmyadmin_user`@`localhost` IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' |
| GRANT ALL PRIVILEGES ON `phpmyadmin_db`.* TO `phpmyadmin_user`@`localhost`                                             |
+------------------------------------------------------------------------------------------------------------------------+
```


### Paso 2
> Instalar `phpmyadmin` desde repositorios

```
sudo apt install phpmyadmin
```

Nos pregunta qué servidor web queremos dejar configurado para `phpmyadmin`, y eligimos `apache2`:
![](https://i.imgur.com/UPcWL2u.png)

Decimos que sí para que configure la base de datos `phpmyadmin_db` que usará `phpmyadmin`:
![](https://i.imgur.com/U8Wvaxo.png)

Especificamos método de conexión a MariaDB por Unix socket:
![](https://i.imgur.com/kzwewbn.png)

Dejamos por defecto el método de autenticación con MariaDB:
![](https://i.imgur.com/QWFVCBG.png)

Base de datos a utilizar por `phpmyadmin`:
![](https://i.imgur.com/CCIU2Md.png)

Usuario para la conexión con esa base de datos:
![](https://i.imgur.com/nfmXomq.png)

Escribimos `1234`, la contraseña del usuario `phpmyadmin_user`. La necesitará para poder acceder y configurar su base de datos:

![](https://i.imgur.com/nQZLFot.png)

Confirmamos la contraseña:

![](https://i.imgur.com/CubnyMu.png)

Termina la instalación


> Comprobar el acceso en /phpmyadmin

Por defecto, tenemos este "error":
![](https://i.imgur.com/PpeW8Wb.png)

Lo escribo entre comillas porque no es un error en sí, sino que apache no tiene funcionando la ejecución de código php.  
Nos está mostrando el `index.php` en crudo.

Para solucionarlo:
```
sudo apt install php
sudo a2enmod php7.4
```

Hacemos Ctrl F5 para recargar ignorando la caché, y ya funciona:
![](https://i.imgur.com/9vV6vwh.png)


### Paso 3
> ¿Se ha creado en el DocumentRoot un directorio que se llama phpmyadmin?

No.

> ¿Cómo es que podemos acceder?

El paquete `phpmyadmin` nos generó el enlace simbólico `phpmyadmin.conf` en `/etc/apache2/conf-available`, que por defecto se habilita en `conf-enabled`.

El fichero original `/etc/phpmyadmin/apache.conf` contiene 2 directivas que hacen que funcione el acceso:
```
Alias /phpmyadmin /usr/share/phpmyadmin

<Directory /usr/share/phpmyadmin>
```
*(he mostrado sólo lo necesario, en el fichero original hay mucho más)*

Lo que exista en `conf-enabled` es configuración global, por lo tanto este Alias y Directory se tendrían en cuenta desde cualquier VirtualHost.


### Paso 5
> Deshabilitar acceso a phpmyadmin

```
sudo a2disconf phpmyadmin
sudo systemctl restart apache2
```

> Comprobar que no se puede acceder

![](https://i.imgur.com/TeGgSE9.png)

> Crear VirtualHost con `ServerName` `basededatos.adrianjaramillo.org`
y que muestre phpmyadmin

Creo `/etc/apache2/sites-available/phpmyadmin.conf` con el siguiente contenido:
```
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.

	ServerName basededatos.adrianjaramillo.org
	DocumentRoot /usr/share/phpmyadmin

        <Directory /usr/share/phpmyadmin>
            Options SymLinksIfOwnerMatch
            DirectoryIndex index.php

            # limit libapache2-mod-php to files and directories necessary by pma
            <IfModule mod_php7.c>
                php_admin_value upload_tmp_dir /var/lib/phpmyadmin/tmp
                php_admin_value open_basedir /usr/share/phpmyadmin/:/usr/share/doc/phpmyadmin/:/etc/phpmyadmin/:/var/lib/phpmyadmin/:/usr/share/php/:/usr/share/javascript/
            </IfModule>

        </Directory>

        # Disallow web access to directories that don't need it
        <Directory /usr/share/phpmyadmin/templates>
            Require all denied
        </Directory>
        <Directory /usr/share/phpmyadmin/libraries>
            Require all denied
        </Directory>

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Lo habilito:
```
sudo a2ensite phpmyadmin.conf
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

Modifico mi `/etc/hosts`:
```
# ej1-phpmyadmin resolutions
192.168.121.96 basededatos.adrianjaramillo.org
```

Muestro que accedo a phpmyadmin desde nombre y funciona:
![](https://i.imgur.com/JJezRyu.png)


### Paso 6
> Acceder a phpmyadmin con el usuario del punto 1 `phpmyadmin_user`

A la derecha vemos que estamos conectados con el usuario requerido:
![](https://i.imgur.com/OdWmih8.png)


> Comprobar que podemos gestionar su base de datos

Muestro que puedo *crear una tabla*:
![](https://i.postimg.cc/zBSfMwFr/phpmyadmin-crear-tabla.gif)

Muestro que puedo *borrar una tabla*:
![](https://i.postimg.cc/76QRqdVM/phpmyadmin-borrar-tabla.gif)
