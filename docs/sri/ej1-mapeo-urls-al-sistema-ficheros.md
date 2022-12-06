# Ejercicio 1: Mapear URL a ubicaciones de un sistema de ficheros

## ENTREGA

### Parte 1
> Mostrar configuración completa del virtualhost

```
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.

        ServerName www.mapeo.com
	    DocumentRoot /srv/mapeo

        RedirectMatch ^/$ /principal

        <Directory /srv/mapeo/principal>
          Options -Indexes
          Options -FollowSymLinks
          Options -MultiViews
        </Directory>

        Alias /principal/documentos /home/vagrant/doc
        <Directory /home/vagrant/doc>
          Options Indexes SymLinksIfOwnerMatch
          Require all granted
        </Directory>

        ErrorDocument 404 "/error/404.html"
        ErrorDocument 403 "/error/403.html"        

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

### Parte 2
> Con F12 de firefox, entrega screenshot para mostrar el redireccionamiento de `www.mapeo.com` a `principal`

![](https://i.imgur.com/HvcheZv.png)

### Parte 3
> Mostrar screenshot en `www.mapeo.com/principal/documentos`

![](https://i.imgur.com/LeUalbf.png)

### Parte 4
> Mostrar screenshot de 404 custom

![](https://i.imgur.com/YF9xncS.png)

> Mostrar screenshot de 403 custom

![](https://i.imgur.com/20fVENt.png)







## EJERCICIOS
### Ejercicio 0
> Crear virtualhost www.mapeo.com con DocumentRoot /srv/mapeo

Creo `/etc/apache2/sites-available/mapeo.conf`:
```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.

        ServerName www.mapeo.com  
        DocumentRoot /srv/mapeo    

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

En `/etc/apache2/apache2.conf` descomento:
```
<Directory /srv/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

Habilito el nuevo VirtualHost:
```
sudo a2ensite mapeo.conf
```

Reinicio apache:
```
sudo systemctl restart apache2
```

En mi máquina añado a `/etc/hosts` lo siguiente:
```
# ej1mapeourls resolutions
192.168.121.8 www.mapeo.com
```

Veo que funciona con un `index.html` de prueba:
![](https://i.imgur.com/aWbl8AC.png)

### Ejercicio 1
> Redireccionar www.mapeo.com a www.mapeo.com/principal, con mensaje de bienvenida

En `/etc/apache2/sites-available/mapeo.conf` añado:
```
RedirectMatch ^/$ /principal
```

Reinicio apache:
```
sudo systemctl restart apache2
```

Compruebo que la redirección funciona:
![](https://i.postimg.cc/HxTjSwsC/redirect-principal.gif)


### Ejercicio 2
> En www.mapeo.com/principal se deniega:
>
>- Listado de contenidos
>- Enlaces simbólicos
>- Negociación de contenidos

La configuración añadida en `/etc/apache2/sites-available/mapeo.conf` sería:
```
<Directory /srv/mapeo/principal>
  Options -Indexes
  Options -FollowSymLinks
  Options -MultiViews
</Directory>
```

Reinicio apache:
```
sudo systemctl restart apache2
```

#### Denegación de listado de contenidos  
Modifico el nombre del `index.html` en `/srv/mapeo/principal` a otro cualquiera.  
Hago esto porque `Indexes` primero busca un `index.html` en el directorio, y si no lo encuentra, intenta listar el contenido.

Muestro que deniega el listado:

![](https://i.imgur.com/PXOV4jL.png)

#### Denegación de enlaces simbólicos  

Creo uno de prueba:
```
sudo ln -s ~/test.txt test.txt
```

Muestro que deniega el acceso:
![](https://i.imgur.com/OsafDdY.png)

#### Denegación de negociación de contenidos

Creo 2 `index.html` de prueba, uno en español y otro en inglés:
```
-rw-r--r-- 1 root root  378 Oct 24 17:45 index.html.en
-rw-r--r-- 1 root root  383 Oct 24 17:44 index.html.es
```

Muestro que deniega el acceso:
![](https://i.imgur.com/LNWfhIN.png)


### Ejercicio 3
> Hacer Alias de `www.mapeo.com/principal/documentos` a `/home/vagrant/doc`  
>
>Se permitirá:
>
>- Indexes
>- SymLinksIfOwnerMatch

En `/home/vagrant/doc` he creado la siguiente estructura de pruebas:
```
lrwxrwxrwx 1 vagrant vagrant   23 Oct 24 18:17 dummy.pdf -> /home/vagrant/dummy.pdf
-rw-r--r-- 1 vagrant vagrant 3028 Feb 24  2017 sample.pdf
```

Vemos que `dummy.pdf` mantiene el mismo owner que el enlace simbólico en el fichero original:
```
vagrant@ej1mapeourls:~$ ls -la
total 56
drwxr-xr-x 5 vagrant vagrant  4096 Oct 24 18:15 .
drwxr-xr-x 3 root    root     4096 Aug 29 17:07 ..
-rw-r--r-- 1 vagrant vagrant   220 Aug  4 20:25 .bash_logout
-rw-r--r-- 1 vagrant vagrant  3526 Aug  4 20:25 .bashrc
drwxr-xr-x 3 vagrant vagrant  4096 Oct 22 11:55 .local
-rw-r--r-- 1 vagrant vagrant   807 Aug  4 20:25 .profile
drwx------ 2 vagrant vagrant  4096 Oct 22 11:46 .ssh
-rw-r--r-- 1 vagrant vagrant   216 Oct 24 18:15 .wget-hsts
drwxr-xr-x 2 vagrant vagrant  4096 Oct 24 18:17 doc
-rw-r--r-- 1 vagrant vagrant 13264 Aug 27  2007 dummy.pdf
-rw-r--r-- 1 vagrant vagrant     7 Oct 24 17:27 test.txt
```

Añado lo siguiente a `/etc/apache2/sites-available/mapeo.conf` para la configuración del Alias:
```
Alias /principal/documentos /home/vagrant/doc
<Directory /home/vagrant/doc>
  Options Indexes SymLinksIfOwnerMatch
  Require all granted
</Directory>
```

Reinicio apache:
```
sudo systemctl restart apache2
```

Muestro que funciona:

![](https://i.imgur.com/9QJIe66.png)

En caso de que el owner entre enlace simbólico y fichero original difiriese, simplemente haría que no se nos mostrase en ese listado, aunque el fichero existiera.

El hecho de que `dummy.pdf` se muestre, es prueba de que `SymLinksIfOwnerMatch` está funcionando.

### Ejercicio 4
> Crear `/srv/mapeo/error`

```
sudo mkdir /srv/mapeo/error
```

> Crear en ese directorio 2 html de error custom

```
-rw-r--r-- 1 root root 2599 Oct 24 22:52 403.html
-rw-r--r-- 1 root root 1367 Oct 24 22:43 404.html
```

> Añadir a `/etc/apache2/sites-available/mapeo.conf` lo siguiente:

```
ErrorDocument 404 "/error/404.html"
ErrorDocument 403 "/error/403.html"
```

> Reiniciar apache:

```
sudo systemctl restart apache2
```

> Entro con una URL inventada para probar el 404

![](https://i.imgur.com/YF9xncS.png)

> Entro a `www.mapeo.com/principal` para probar el 403, ya que tenía el Indexes y el MultiViews prohibido

![](https://i.imgur.com/20fVENt.png)
