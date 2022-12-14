---
title: "Práctica servicios: Ejecución de scripts PHP y Python. Rendimiento"
---

# 1. Ejecución de scripts PHP

Las configuraciones que vamos a realizar son las siguientes:

* mod_php + apache2
* PHP-FPM (socket unix) + apache2
* PHP-FPM (socket TCP) + apache2
* PHP-FPM (socket unix) + nginx
* PHP-FPM (socket TCP) + nginx

Sobre cada una de de ellas, tendremos que hacer una serie de acciones

## Combinación 1: mod_php + apache2

### **Realiza la configuración indicada y muestra una comprobación (con phpinfo) donde se vea la configuración actual**
* Instalamos el módulo
```
sudo apt install libapache2-mod-php
```

* Reiniciamos apache
```
sudo systemctl restart apache2.service
```

* Comprobamos que el módulo esté cargado y funcionando
```
sudo apache2ctl -M | grep php
```
Aquí aparece el módulo
```
php7_module (shared)
```

* He aprovechado el document root del virtualhost por defecto, para crear ahí un `info.php` de prueba  
```
/var/www
└── /var/www/html
    ├── /var/www/html/index.html
    └── /var/www/html/info.php
```

* Muestro que el contenido de info.php se carga correctamente
![](https://i.imgur.com/ASpm9di.png)


### **Explica brevemente la modificación en los ficheros de configuración para la opción actual**








### **Mostrar el funcionamiento con esta configuración de WordPress**

* Podemos descargarnos WordPress [aquí](https://wordpress.org/latest.tar.gz). Cuando descomprimamos el paquete, podremos el document root que existe por defecto, y renombrar el directorio resultante de Wordpress a `/var/www/html`. Así estaremos aprovechando el virtualhost que hay por defecto, y podremos iniciar Wordpress sin más cambios

* Nos faltaría tener una base de datos
```
sudo apt install mariadb-server
```

* Nos logeamos como root en mariadb
```
sudo mariadb
```

* Creamos la base de datos para este Wordpress específico
```
CREATE DATABASE wordpress_rend DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

* Con lo siguiente se creará directamente el usuario con el que accederemos a la base de datos, y además se le concederán todos los privilegios sobre ésta
```
GRANT ALL ON wordpress_rend.* TO 'wordpress_rend_user'@'localhost' IDENTIFIED BY '1234';
```

* Aplicamos los permisos
```
FLUSH PRIVILEGES;
```

* Nos salimos de mariadb, e instalamos los siguientes paquetes php que necesitaremos (ya sea porque son requeridos por Wordpress, o porque permiten la conexión de php con mariadb)
```
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip php-mysql
```

* Reiniciamos apache para que se apliquen los cambios
```
sudo systemctl restart apache2
```

* Una vez hecho todo esto, Wordpress ya cargará correctamente, y sólo tendremos que ir siguiendo los pasos para completar la instalación.
> Usuario administrador: rootadri  
> Contraseña: 1234

* Muestro que WordPress funciona con ésta combinación de mod_php + apache2
![](https://i.imgur.com/n9NZDWD.png)


### **Modifica la configuración de PHP para aumentar el tamaño de los ficheros que podemos subir**
Para empezar, es vital saber que los parámetros de configuración php que están afectando a nuestro Wordpress actualmente, se encuentran en `/etc/php/7.3/apache2/php.ini`. ¿Por qué? Pues porque al estar funcionando con mod_php, es Apache quien realmente está ejecutando el código php.

Con respecto a cómo php maneja los tamaños de las subidas de ficheros...etc, existen 3 parámetros íntimamente relacionados que influyen en este funcionamiento:
* _upload_max_filesize_ (tamaño máximo permitido para la subida de ficheros)
* _post_max_size_ (tamaño máximo de datos POST que php va a aceptar)
* _memory_limit_ (cantidad máxima de memoria que un script va a poder consumir)

Algo que se recomienda sobre estos 3 parámetros, es que sus valores se encuentren ajustados de forma escalonada.  
Para entender esto mejor, mostraré mis valores en `/etc/php/7.3/apache2/php.ini` por defecto:
```
upload_max_filesize = 2M
post_max_size = 8M
memory_limit = 128M
```
Ahora mismo, sólo podríamos subir ficheros de como máximo 2MB, como efectivamente muestro aquí abajo: 
![](https://i.imgur.com/N2uyLuU.png)

> Aunque ahora mismo nos podríamos preguntar...¿por qué existe una diferencia entre `upload_max_filesize` y `post_max_size`?  
Pues porque siempre existe la posibilidad de que en una subida de un fichero, no sólo se encuentren los datos del propio fichero en el body de la petición POST, sino que puede haber algo más de información adicional en texto.  
Es por ésta razón que siempre se deja este margen de seguridad, para tener en cuenta el tamaño máximo del fichero a poder subir + lo que pueda haber.

Dicho todo esto, pasaré a modificar los valores de los que he estado hablando. De los 3 que hay, simplemente necesito modificar 2 de ellos. Mi configuración se quedaría de la siguiente manera:
```
upload_max_filesize = 10M
post_max_size = 20M
```

Para aplicar los cambios, tendremos que reiniciar Apache
```
sudo systemctl restart apache2.service
```
Hacemos esto porque, como es Apache quien ejecuta el código php con su propio módulo, con una configuración específica de php para él, tendremos que reiniciarlo, para que vuelva a leer la configuración que hemos modificado.

Ahora sí, el tamaño máximo de archivo que podremos subir es de 10MB, tal y como hemos configurado
![](https://i.imgur.com/4v6NVb2.png)




### **Realiza varias pruebas (al menos 5) de rendimiento sobre la configuración actual y quedáte con una media de las peticiones respondidas por segundo. Vamos a realizar pruebas con 200 peticiones concurrentes. Nota: Recuerda reinciar el servidor web entre prueba y prueba (después de cada test de ab, para que se reinicien el número de procesos de apache manejando peticiones)**

Comando que vamos a realizar para las pruebas
```
ab -t 10 -c 200 -k http://www.wordpress-rend-adri.com/index.php
```

> Como habrás podido notar, he cambiado la dirección de acceso a `www.wordpress-rend-adri.com`, en vez de estar accediendo por la 192.168.1.116. He tenido que hacer este cambio porque realicé la instalación de Wordpress accediendo desde la IP dicha, y entonces todas las URL se guardaron en la base de datos siguiendo ese patrón de IP. Por este motivo, al intentar acceder a cualquier enlace estando en clase, no funciona, porque se intentan las conexiones con una IP que no existe en esa red.  
Por lo tanto encontré el siguiente comando (utilidad que nos aporta la herramienta wp-cli), con el que se reemplazarán en la base de datos todas las URLs que tuvieran la IP `192.168.1.116`, por el nombre `www.wordpress-rend-adri.com`.  
>```
>sudo wp search-replace http://192.168.1.116 http://www.wordpress-rend-adri.com
>```
>De esta manera estaríamos arreglando el problema de acceso en el instituto a Wordpress, porque al acceder por nombre, ya sólo tendríamos que cambiar la IP en nuestro `/etc/hosts` cuando cambiásemos de red para que la conexión funcionase.  
Tras ejecutar el comando anterior, al recargar la página en firefox limpiando la caché (ctrl+shift+r), las URLs en nuestro CMS habrán cambiado, y ya podremos navegar por él sin problemas


* Prueba 1
```
Requests per second:    51.19 [#/sec] (mean)
```

* Prueba 2
```
Requests per second:    242.43 [#/sec] (mean)
```

* Prueba 3
```
Requests per second:    48.49 [#/sec] (mean)
```

* Prueba 4
```
Requests per second:    244.52 [#/sec] (mean)
```

* Prueba 5
```
Requests per second:    47.34 [#/sec] (mean)
```


* Media de peticiones respondidas por segundo: **126.794**






## Combinación 2: PHP-FPM (socket unix) + apache2

### **Realiza la configuración indicada y muestra una comprobación (con phpinfo) donde se vea la configuración actual**

* Lo primero que tenemos que hacer es cambiar el modo mpm en el que está funcionando apache a event. Para ello, es obligatorio que deshabilitemos PRIMERO mod_php.

>**¿Por qué?** Porque al tenerlo funcionando por la configuración anterior que hicimos, y al depender del módulo prefork, es necesario que lo deshabilitemos primero antes de cambiar el módulo prefork por event.

```
sudo a2dismod php7.3
```

* Después de haber hecho esto, podemos deshabilitar prefork, y activar event
```
sudo a2dismod mpm_prefork
```
```
sudo a2enmod mpm_event
```

* Reiniciamos apache para que se apliquen los cambios
```
sudo systemctl restart apache2.service
```

* Comprobamos que tenemos event cargado correctamente
```
sudo apache2ctl -V
```
Voy a mostrar SÓLO la información relevante de la salida de éste comando  
```
Server MPM:     event
  threaded:     yes (fixed thread count)
    forked:     yes (variable process count)
```

* Una vez hecho todo esto, si intentamos navegar de nuevo por Wordpress, directamente no cargará nada. Esto sucede porque antes hemos deshabilitado mod_php, y por lo tanto ya no hay nada que se encargue de manejar el código php. En éste punto, es cuando tenemos que instalar php-fpm:
```
sudo apt install php-fpm
```

* Lo siguiente que tendríamos que configurar, es su funcionamiento mediante socket unix. Por suerte por defecto, ya viene configurado de ésta manera. El fichero `/etc/php/7.3/fpm/pool.d/www.conf`, se utiliza para configurar php-fpm, y dentro de ese fichero, viene una línea muy importante:
```
listen = /run/php/php7.3-fpm.sock
```
Esta línea está indicando la dirección donde php-fpm está aceptando peticiones. En este caso, como estamos usando socket, viene indicada la dirección a éste.

* Habilitamos unos módulos necesarios en apache, y reiniciamos
```
sudo a2enmod proxy proxy_fcgi
```
```
sudo systemctl restart apache2.service
```
Estos módulos son necesarios porque apache tiene que ser capaz de redirigir las peticiones a ficheros php, a php-fpm para que las procese, y le devuelva un resultado.

* Después de todo ésto, lo único que tendremos que hacer es añadir el siguiente bloque de configuración a nuestro virtualhost por defecto que estamos usando
```
	<FilesMatch \.php$>
		# 2.4.10+ can proxy to unix socket
	    SetHandler "proxy:unix:/run/php/php7.3-fpm.sock|fcgi://localhost/"

		# Else we can just use a tcp socket:
		# SetHandler "proxy:fcgi://127.0.0.1:9000"
    </FilesMatch>
```
Con ésta configuración simplemente lo que estamos diciendo es que, cuando se estén haciendo peticiones sobre ficheros que terminen en ".php", se manden mediante el socket en el que está escuchando php-fpm. Por lo tanto al final, php-fpm será capaz de procesar esas peticiones, ejecutando todo el código php que necesite ejecutar, y devolverá los resultados a apache. 

Al final, para que todo termine de funcionar, tenemos que reiniciar apache una última vez
```
sudo systemctl restart apache2.service
```

* Para terminar, mostraré la página de phpinfo

[![](https://i.imgur.com/Gk3EKU0.png)](https://i.imgur.com/Gk3EKU0.png)






### **Explica brevemente la modificación en los ficheros de configuración para la opción actual**



### **Mostrar el funcionamiento con esta configuración de WordPress**

[![](https://i.imgur.com/lDjaCsw.png)](https://i.imgur.com/lDjaCsw.png)

> No he tenido que hacer ningún cambio para que Wordpress funcionase de nuevo. Conque se active php-fpm, es suficiente, porque lo único que sucedía es que no había nada que pudiera procesar las peticiones php


### **Modifica la configuración de PHP para aumentar el tamaño de los ficheros que podemos subir**


### **Realiza varias pruebas (al menos 5) de rendimiento sobre la configuración actual y quedáte con una media de las peticiones respondidas por segundo. Vamos a realizar pruebas con 200 peticiones concurrentes. Nota: Recuerda reinciar el servidor web entre prueba y prueba (después de cada test de ab, para que se reinicien el número de procesos de apache manejando peticiones)**

Comando que vamos a realizar para las pruebas
```
ab -t 10 -c 200 -k http://www.wordpress-rend-adri.com/index.php
```
























## Combinación 3: PHP-FPM (socket TCP) + apache2


## Combinación 4: PHP-FPM (socket unix) + nginx


## Combinación 5: PHP-FPM (socket TCP) + nginx




## ¿Qué combinación ha respondido más peticiones por segundo según vuestras pruebas?





















# 2. Aumento de rendimiento en la ejecución de scripts PHP








# 3. Ejecución de scripts Python
Instalar django-cms en lugar de mezzanine







# :fountain: Fuentes :fountain:
<https://kinsta.com/blog/wordpress-maximum-upload-file-size/#>
<https://www.cyberciti.biz/tips/howto-performance-benchmarks-a-web-server.html>
<https://serversforhackers.com/c/apache-and-php-fpm>


