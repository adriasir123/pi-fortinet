# Ejercicio 3: Ejecución de PHP con PHP-FPM

## ENTREGA

### Parte 1
> Captura para comprobar que se están ejecutando procesos php-fpm

Mediante `top`:

![](https://i.imgur.com/lZJp2Mf.png)

Mediante `systemctl`:

![](https://i.imgur.com/fbeaLhe.png)

### Parte 2
> Configurar Apache para que utilice php-fpm

Activo el módulo:
```
sudo a2enmod proxy_fcgi
```

Activo la configuración php-fpm global:
```
sudo a2enconf php7.4-fpm
```

Reinicio apache:
```
sudo systemctl restart apache2
```

### Parte 3
> Captura de `info.php` donde se vea que se utiliza php-fpm

![](https://i.imgur.com/F1Fn8CE.png)

### Parte 4
> Captura de phpBB funcionando

![](https://i.imgur.com/UQfxX20.png)

> Captura de ConcreteCMS funcionando

![](https://i.imgur.com/dIJPpFK.png)

### Parte 5
> Hacer que php-fpm escuche por socket TCP

Modificar `/etc/php/7.4/fpm/pool.d/www.conf`:
```
listen = 127.0.0.1:9000
```

Reiniciar php-fpm:
```
sudo systemctl restart php7.4-fpm.service
```

> Hacer que Apache se comunique con php-fpm por socket TCP

Modificar `/etc/apache2/conf-available/php7.4-fpm.conf`:
```
<FilesMatch ".+\.ph(ar|p|tml)$">
		SetHandler "proxy:fcgi://127.0.0.1:9000/"
</FilesMatch>
```

Reiniciar Apache:
```
sudo systemctl restart apache2.service
```

### Parte 6
> Indicar el fichero modificado para cambiar el `memory_limit`

`/etc/php/7.4/fpm/php.ini`

> Captura de `info.php` donde se vea el cambio

![](https://i.imgur.com/eg7kHVg.png)





## REALIZACIÓN

Los siguientes pasos se realizan sobre el escenario de la práctica "Implantación de aplicaciones web php".

### Paso 1
> Desinstalar el módulo de apache que permite ejecutar PHP

```
sudo apt purge libapache2-mod-php7.4
```

Sabemos que ha surtido efecto porque al probar nuestra web, no funciona y ahora nos devuelve código php en crudo:

![](https://i.imgur.com/ez9QBDR.png)

### Paso 2
> Instalar php-fpm

```
sudo apt install php7.4-fpm
```

### Paso 3
> Configurar Apache para que utilice php-fpm globalmente *(para todos los VirtualHost)*

Activo el módulo:
```
sudo a2enmod proxy_fcgi
```

Activo la configuración php-fpm global:
```
sudo a2enconf php7.4-fpm
```

Reinicio apache:
```
sudo systemctl restart apache2
```

### Paso 4
> Acceder a un `info.php` para comprobar que se está usando php-fpm

En el DocumentRoot de phpBB me descargo un `info.php`:
```
sudo wget https://gist.githubusercontent.com/carlosomarsuarez/9c7860d74535f4f0960e/raw/ac8170c4142933a48fd7ed8b88eb9d4f190ec32c/info.php
```

Accedo al fichero:

![](https://i.imgur.com/F1Fn8CE.png)

> Comprobar que phpBB sigue funcionando

![](https://i.imgur.com/UQfxX20.png)

> Comprobar que ConcreteCMS sigue funcionando

![](https://i.imgur.com/dIJPpFK.png)

### Paso 5
> Hacer que php-fpm escuche por socket TCP

Modificar `/etc/php/7.4/fpm/pool.d/www.conf`:
```
listen = 127.0.0.1:9000
```

Reiniciar php-fpm:
```
sudo systemctl restart php7.4-fpm.service
```

> Hacer que Apache se comunique con php-fpm por socket TCP

Modificar `/etc/apache2/conf-available/php7.4-fpm.conf`:
```
<FilesMatch ".+\.ph(ar|p|tml)$">
		SetHandler "proxy:fcgi://127.0.0.1:9000/"
</FilesMatch>
```

Reiniciar Apache:
```
sudo systemctl restart apache2.service
```

### Paso 6
> Comprobar que phpBB sigue funcionando

![](https://i.imgur.com/F5wfmO4.png)

> Comprobar que ConcreteCMS sigue funcionando

![](https://i.imgur.com/zEtq3QL.png)

### Paso 7
> Cambiar el `memory_limit` de php-fpm a 256M

Modificar `/etc/php/7.4/fpm/php.ini`:
```
memory_limit = 256M
```

Reiniciar php-fpm:
```
sudo systemctl restart php7.4-fpm.service
```

Comprobamos que el cambio haya tomado efecto mirando el `info.php`:

![](https://i.imgur.com/eg7kHVg.png)
