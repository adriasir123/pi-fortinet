# Ejercicio 2: Instalación de BookMedik

## ENTREGA

### Parte 1
> Captura del VirtualHost

![](https://i.imgur.com/N5V1lLp.png)


### Parte 2
> Contenido del fichero `core/controller/Database.php`

```
<?php
class Database {
	public static $db;
	public static $con;
	function Database(){
		$this->user="bookmedik_user";$this->pass="1234";$this->host="localhost";$this->ddbb="bookmedik";
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


### Parte 3
> Captura de bookmedik, después de login

![](https://i.imgur.com/t0KKeOw.png)


### Parte 4
> Indicar ruta completa al fichero donde se modifica `memory_limit`

`/etc/php/7.4/apache2/php.ini`

> Captura de `info.php` donde se vea el cambio

![](https://i.imgur.com/gKCxD4Z.png)



## REALIZACIÓN

### Paso 0
> Instalar `mariadb-server`

```
sudo apt update
sudo apt install mariadb-server
```


### Paso 1
> Descargar `schema.sql` *(copia de seguridad de la BD)* del repositorio <https://github.com/evilnapsis/bookmedik>

En el repositorio, abrir la versión "raw" de `schema.sql`.  

Después, descargar este fichero usando la URL en modo raw:
```
wget https://raw.githubusercontent.com/evilnapsis/bookmedik/master/schema.sql
```

> Importar la copia de seguridad `schema.sql`

```
sudo mysql -u root < schema.sql
```

Vemos que se ha creado la base de datos `bookmedik`:
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| bookmedik          |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```

> Crear usuario con privilegios sobre `bookmedik`

```
CREATE USER 'bookmedik_user' IDENTIFIED BY '1234';

GRANT USAGE ON *.* TO 'bookmedik_user'@localhost IDENTIFIED BY '1234';

GRANT ALL privileges ON `bookmedik`.* TO 'bookmedik_user'@localhost;

FLUSH PRIVILEGES;
```


### Paso 2
> Instalar apache2 y php

```
sudo apt install apache2
sudo apt install php php-mysql
```

> Crear VirtualHost con `ServerName` `bookmedik.adrianjaramillo.org`

```
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.

	ServerName bookmedik.adrianjaramillo.org
	DocumentRoot /var/www/bookmedik

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
sudo a2ensite bookmedik.conf
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

Modifico mi `/etc/hosts`:
```
# ej2-bookmedik resolutions
192.168.121.67 bookmedik.adrianjaramillo.org
```

> Clonar <https://github.com/evilnapsis/bookmedik> en el DocumentRoot `/var/www/bookmedik`

```
git clone https://github.com/evilnapsis/bookmedik .
```

Este clonado no nos crea un directorio donde mete el repositorio, sino que directamente clona los contenidos al directorio actual.

La única condición para que este clonado funcione, es que el directorio destino esté vacío.


### Paso 3
> Modifica `core/controller/Database.php` con los datos de acceso a la BD creada en el paso 1

Este es el fragmento modificado:
```
function Database(){
	$this->user="bookmedik_user";$this->pass="1234";$this->host="localhost";$this->ddbb="bookmedik";
}
```


### Paso 4
> Acceder a `bookmedik.adrianjaramillo.org` con usuario admin y contraseña admin

![](https://i.imgur.com/t0KKeOw.png)


### Paso 5
> Cambia el `memory_limit` a 256M en `php.ini`

Antes de nada, tenemos que averiguar dónde se encuentra nuestro fichero `php.ini`.  

**¡OJO!** Hay 2 ficheros `php.ini`, así que tenemos que encontrar el usado por apache, *EL DE CLI NO*.

Me descargo un fichero `info.php` de prueba en `/var/www/bookmedik` para encontrar la ruta al `php.ini` que se está usando:
```
sudo wget https://gist.githubusercontent.com/SyntaxC4/5648247/raw/94277156638f9c309f2e36e19bff378ba7364907/info.php
```

Aquí muestro la ruta:
![](https://i.imgur.com/6tzGTjv.png)

En `/etc/php/7.4/apache2/php.ini` modificamos tal que así:
```
; Maximum amount of memory a script may consume
; http://php.net/memory-limit
memory_limit = 256M
```

Reiniciamos apache
```
sudo systemctl restart apache2
```

Muestro que el valor `memory_limit` es ahora el correcto:
![](https://i.imgur.com/gKCxD4Z.png)
