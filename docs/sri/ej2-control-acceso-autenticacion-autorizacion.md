# Ejercicio 2: Control de acceso, autenticación y autorización

## ENTREGA

### Parte 1
> Captura accediendo a `departamentos.iesgn.org/intranet` desde nuestro host

![](https://i.imgur.com/AbQcx2L.png)

> Captura accediendo a `departamentos.iesgn.org/intranet` desde cliente

![](https://i.imgur.com/yO8mo59.png)

### Parte 2
> Captura accediendo a `departamentos.iesgn.org/internet` desde nuestro host

![](https://i.imgur.com/Uc4khoh.png)

> Captura accediendo a `departamentos.iesgn.org/internet` desde cliente

![](https://i.imgur.com/DxwK452.png)

### Parte 3
> Mostrar que la autenticación básica funciona

![](https://i.postimg.cc/fRDHyYRM/autenticacion-basica-apache.gif)

### Parte 4
> Captura con las cabeceras HTTP donde se vea la autenticación básica

![](https://i.imgur.com/wK84JnE.png)

Se manda una cabecera `Authorization` en la petición, con el mensaje `dmlwOnZpcA==` en texto plano.

Con cualquier decodificador online de Base64 podemos averiguar lo que significa:

![](https://i.imgur.com/9FDigJG.png)

Pues bien, ahí tenemos el usuario y la contraseña que he introducido durante la autenticación básica.

### Parte 5
> Mostrar que la autenticación digest funciona

Pruebo el acceso con `nodirectivo1`:

![](https://i.postimg.cc/CMvXMXwM/auth-digest-nodirectivo.gif)

Pruebo el acceso con `directivo1`:

![](https://i.postimg.cc/j2FRzjNZ/auth-digest-directivo.gif)

### Parte 6
> Captura con las cabeceras HTTP donde se vea la autenticación digest

![](https://i.imgur.com/Kd06SYn.png)

### Parte 7
> Mostrar el funcionamiento del ejercicio 4

Prueba de acceso desde cliente:

![](https://i.postimg.cc/W1sWPcy1/acceso-secreto-intranet.gif)

Prueba de acceso desde mi máquina:

![](https://i.postimg.cc/NjP1PhqV/acceso-secreto-internet.gif)

> Entregar la configuración realizada

Modifico `/etc/apache2/sites-available/departamentos.conf`:
```
<Directory "/var/www/departamentos/secreto">
  <RequireAny>
    <RequireAll>
      Require ip 10.0.0
    </RequireAll>
    <RequireAll>
      Require ip 192.168.121
      AuthType Digest
      AuthName "directivos"
      AuthUserFile /etc/apache2/.htpasswd
      Require valid-user  
    </RequireAll>
  </RequireAny>
</Directory>
```

Reinicio Apache:
```
sudo systemctl restart apache2.service
```









## Preliminares

### Control de acceso
> ¿Control de acceso por defecto de 000-default?

En `000-default.conf` no hay control de acceso definido, pero como el DocumentRoot está en `/var/www/html`, le afecta la siguiente directiva de `apache2.conf`:
```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```


## EJERCICIOS

### Ejercicio 0
> Definir escenario Vagrant:
>
>- Servidor:
    - Interfaz eth0 (NAT) actuando como pública
    - Interfaz privada (veryisolated)
- Cliente conectado a la red privada

```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :ej2controlacceso do |ej2controlacceso|
    ej2controlacceso.vm.box = "debian/bullseye64"
    ej2controlacceso.vm.hostname = "ej2controlacceso"
    ej2controlacceso.vm.network :private_network,
      :libvirt__network_name => "ej2controlaccesonet",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.2",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :cliente do |cliente|
    cliente.vm.box = "debian/bullseye64"
    cliente.vm.hostname = "cliente"
    cliente.vm.network :private_network,
      :libvirt__network_name => "ej2controlaccesonet",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.3",
      :libvirt__forward_mode => "veryisolated"
  end

end
```

> Crear VirtualHost `departamentos.iesgn.org`

```
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.

	ServerName departamentos.iesgn.org
	DocumentRoot /var/www/departamentos

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

### Ejercicio 1
> A `departamentos.iesgn.org/intranet`:
>
>- Se permite acceso desde cliente
>- Se deniega acceso desde nuestra máquina

Modificar `/etc/apache2/sites-available/departamentos.conf`:
```
<Directory /var/www/departamentos/intranet>
  Require ip 10.0.0
</Directory>
```

Reiniciar Apache:
```
sudo systemctl restart apache2.service
```

Prueba de acceso desde mi máquina:

![](https://i.imgur.com/AbQcx2L.png)

Prueba de acceso desde cliente:

![](https://i.imgur.com/yO8mo59.png)

He usado "lynx".


> A `departamentos.iesgn.org/internet`:
>
>- Se permite acceso desde nuestra máquina
>- Se deniega acceso desde cliente

Modificar `/etc/apache2/sites-available/departamentos.conf`:
```
<Directory /var/www/departamentos/internet>
  Require ip 192.168.121       
</Directory>
```

Reiniciar Apache:
```
sudo systemctl restart apache2.service
```

Prueba de acceso desde mi máquina:

![](https://i.imgur.com/Uc4khoh.png)

Prueba de acceso desde cliente:

![](https://i.imgur.com/DxwK452.png)


### Ejercicio 2
> Autenticación básica en `departamentos.iesgn.org/secreto`

Creo el fichero `htpasswd`:
```
sudo htpasswd -c /etc/apache2/.htpasswd vip
```

He añadido un usuario "vip" con contraseña "vip".

Comprobamos que se ha creado `htpasswd` y sus contenidos:
```
vagrant@ej2controlacceso:/etc/apache2$ cat .htpasswd
vip:$apr1$p6ga61LF$dl2YMKRkVM7GW8AuRUsIN/
```

Modifico `/etc/apache2/sites-available/departamentos.conf`:
```
<Directory "/var/www/departamentos/secreto">
  AuthType Basic
  AuthName "Acceso para VIPs"
  AuthUserFile /etc/apache2/.htpasswd
  Require valid-user
</Directory>
```

Reinicio Apache:
```
sudo systemctl restart apache2.service
```

Pruebo que funciona:

![](https://i.postimg.cc/fRDHyYRM/autenticacion-basica-apache.gif)


> Comprobar las cabeceras HTTP durante la autenticación básica ¿Cómo se manda la contraseña entre el cliente y el servidor?

![](https://i.imgur.com/wK84JnE.png)

Se manda una cabecera `Authorization` en la petición, con el mensaje `dmlwOnZpcA==` en texto plano.

Con cualquier decodificador online de Base64 podemos averiguar lo que significa:

![](https://i.imgur.com/9FDigJG.png)

Pues bien, ahí tenemos el usuario y la contraseña que he introducido durante la autenticación básica.


### Ejercicio 3
> Modificar la autenticación a tipo digest, y hacer que `departamentos.iesgn.org/secreto` sólo sea accesible por usuarios del grupo directivos

Sobreescribo el antiguo fichero `htpasswd` añadiendo un usuario directivo:
```
sudo htdigest -c /etc/apache2/.htpasswd directivos directivo1
```

Añado también un usuario no directivo:
```
sudo htdigest /etc/apache2/.htpasswd nodirectivos nodirectivo1
```

Ambos usuarios tienen como contraseña "1234".

Ahora el fichero `.htpasswd` luce de la siguiente manera:
```
directivo1:directivos:4fdd1584741ce8cc0d722a82af5d2a1f
nodirectivo1:nodirectivos:305a303414ab365e1637a8f26f2c30cc
```

Habilito el módulo necesario:
```
sudo a2enmod auth_digest
```

Modifico `/etc/apache2/sites-available/departamentos.conf`:
```
<Directory "/var/www/departamentos/secreto">
  AuthType Digest
  AuthName "directivos"
  AuthUserFile /etc/apache2/.htpasswd
  Require valid-user
</Directory>
```

Reinicio Apache:
```
sudo systemctl restart apache2.service
```

Pruebo el acceso con `nodirectivo1`:

![](https://i.postimg.cc/CMvXMXwM/auth-digest-nodirectivo.gif)

Pruebo el acceso con `directivo1`:

![](https://i.postimg.cc/j2FRzjNZ/auth-digest-directivo.gif)


> Comprobar las cabeceras HTTP durante la autenticación digest ¿Cómo funciona esta autenticación?

![](https://i.imgur.com/Kd06SYn.png)

Ahora en la cabecera `Authorization` de la petición, se manda un campo `response` que contiene las credenciales enviadas encriptadas.

No hay manera de sacar la contraseña en texto plano a partir de ese mensaje encriptado, ya que el sistema se hizo así intencionamente para que no fuese posible.


### Ejercicio 4
> Para acceder a `departamentos.iesgn.org/secreto`:
>
>- No pide autenticación desde la intranet
>- Pide autenticación desde nuestro host

Modifico `/etc/apache2/sites-available/departamentos.conf`:
```
<Directory "/var/www/departamentos/secreto">
  <RequireAny>
    <RequireAll>
      Require ip 10.0.0
    </RequireAll>
    <RequireAll>
      Require ip 192.168.121
      AuthType Digest
      AuthName "directivos"
      AuthUserFile /etc/apache2/.htpasswd
      Require valid-user  
    </RequireAll>
  </RequireAny>
</Directory>
```

Reinicio Apache:
```
sudo systemctl restart apache2.service
```

Prueba de acceso desde cliente:

![](https://i.postimg.cc/W1sWPcy1/acceso-secreto-intranet.gif)

Prueba de acceso desde mi máquina:

![](https://i.postimg.cc/NjP1PhqV/acceso-secreto-internet.gif)
