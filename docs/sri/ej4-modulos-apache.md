# Ejercicio 4: Módulos en Apache

## mod_userdir
Este módulo permite que cada usuario aloje su web en un directorio (`public_html` por defecto).

### Entrega

#### Parte 1
> Mostrar la configuración de `mod_userdir` donde se vea el cambio de nombre del directorio `public_html` por `sitio_web`

Modifico `/etc/apache2/mods-available/userdir.conf`:
```
<IfModule mod_userdir.c>
	UserDir sitio_web
	UserDir disabled root

	<Directory /home/*/sitio_web>
		AllowOverride FileInfo AuthConfig Limit Indexes
		Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
		Require method GET POST OPTIONS
	</Directory>
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

#### Parte 2
> Captura donde se vea el acceso a la página personal de `vagrant`

![](https://i.imgur.com/Z7IW405.png)






### Ejercicios

#### Ejercicio 1
> Activar el módulo

```
sudo a2enmod userdir
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

> Comprobar su funcionamiento

Creo el directorio necesario para el usuario `vagrant`:
```
mkdir ~/public_html
```

Me bajo un `index.html` de prueba en ese directorio:
```
wget https://gist.githubusercontent.com/chrisvfritz/bc010e6ed25b802da7eb/raw/18eaa48addae7e3021f6bcea03b7a6557e3f0132/index.html
```

Lo edito un poco para que se adapte a mi escenario.

Muestro que puedo acceder:

![](https://i.imgur.com/mgcRneZ.png)


#### Ejercicio 2
> Mostrar la configuración por defecto de `mod_userdir`

`/etc/apache2/mods-available/userdir.conf`:
```
<IfModule mod_userdir.c>
	UserDir public_html
	UserDir disabled root

	<Directory /home/*/public_html>
		AllowOverride FileInfo AuthConfig Limit Indexes
		Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
		Require method GET POST OPTIONS
	</Directory>
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

#### Ejercicio 3
> Cambiar el nombre de directorio `public_html` por `sitio_web`

Modifico `/etc/apache2/mods-available/userdir.conf`:
```
<IfModule mod_userdir.c>
	UserDir sitio_web
	UserDir disabled root

	<Directory /home/*/sitio_web>
		AllowOverride FileInfo AuthConfig Limit Indexes
		Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
		Require method GET POST OPTIONS
	</Directory>
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Renombro el antiguo directorio con el nuevo nombre:
```
mv public_html/ sitio_web
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

#### Ejercicio 4
> Mostrar que la web sigue funcionando después del cambio de nombre al directorio

![](https://i.imgur.com/Z7IW405.png)







## mod_rewrite

### Entrega

#### Parte 1
> Entregar la regla de reescritura configurada

Creo el fichero `/var/www/html/php/.htaccess` con el siguiente contenido:
```
Options FollowSymLinks
RewriteEngine On
RewriteBase /php/
RewriteRule ^([A-Za-z]+)/([0-9]+)$ index.php?monto=$2&pais=$1
```

#### Parte 2
> Captura donde se vea el acceso a `www.conversordemoneda.com/php/moneda/cantidad`

![](https://i.imgur.com/oIpBZ8v.png)








### Ejercicios

#### Ejercicio 1
> Crear directorio `php` en el `DocumentRoot` por defecto

```
sudo mkdir /var/www/html/php
```

#### Ejercicio 2
> Añadir el conversor de monedas al directorio `php`

Creo `/var/www/html/php/index.php` con el siguiente contenido:
```
<!DOCTYPE html>
<html lang="es">  
  <head>    
    <title>Conversor de Monedas</title>    
    <meta charset="UTF-8">
  </head>  
  <body>    
	<form action="index.php" method="get">
	   	<input type="text" size="30" name="monto" /><br/>
		<select name="pais">
			<option name="Dolar">Dolar</option>
			<option name="Libra">Libra</option>
			<option name="Yen">Yen</option>
		</select>
	    <input type="submit" value="convertir" />
	   </form>
	<?php
		// averiguamos si se ha introducido un dinero
		if (isset($_GET['monto'])) {
		  define ("cantidad", $_GET['monto']);
		} else {
	 	  define ("cantidad", 0);
		}
		if($_GET){
		// definimos los países
		$tasacambios = array ("Libra"=>0.86,"Dolar"=>1.34,"Yen"=>103.56);
		// imprimimos el monto ingresado
		echo "<b>".cantidad." euros</b><br/> ".$_GET["pais"]." = ".cantidad*$tasacambios[$_GET["pais"]];
		}
	   ?>
	</body>
	</html>
```

#### Ejercicio 3
> Modificar el VirtualHost por defecto para que se acceda por nombre

Modifico `/etc/apache2/sites-available/000-default.conf`:
```
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.

	ServerName www.conversordemoneda.com
	DocumentRoot /var/www/html

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

Reinicio Apache:
```
sudo systemctl restart apache2
```

Modifico mi `/etc/hosts`:
```
# ej4-modulos-apache
192.168.121.204 www.conversordemoneda.com
```

#### Ejercicio 4
> Hacer que Apache funcione con php

```
sudo apt install php
```

#### Ejercicio 5
> Probar que el conversor funciona, primero, sin utilizar rewrite

Hago la prueba con: `www.conversordemoneda.com/php/index.php?monto=100&pais=Libra`:

![](https://i.imgur.com/eXM5p9h.png)

#### Ejercicio 6
> Configurar `mod_rewrite` para que modificando un `.htaccess`, se pueda acceder a `www.conversordemoneda.com/php/moneda/cantidad`  *(moneda=Dolar,Libra,Yen cantidad=euros a convertir)*

Activo el módulo:
```
sudo a2enmod rewrite
```

Habilitar ficheros `.htaccess` en `/etc/apache2/apache2.conf` para el VirtualHost por defecto:
```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
```

Creo el fichero `/var/www/html/php/.htaccess` con el siguiente contenido:
```
Options FollowSymLinks
RewriteEngine On
RewriteBase /php/
RewriteRule ^([A-Za-z]+)/([0-9]+)$ index.php?monto=$2&pais=$1
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

Muestro que funciona:

![](https://i.imgur.com/oIpBZ8v.png)
