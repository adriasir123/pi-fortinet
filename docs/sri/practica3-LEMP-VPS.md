# Práctica : Instalación de un servidor LEMP en nuestra VPS

## Instalación

### Ejercicio 1
> Instalar Nginx

```
sudo apt update
sudo apt install curl gnupg2 ca-certificates lsb-release
sudo apt install nginx
```

### Ejercicio 2
> Instalar servidor MariaDB

```
sudo apt install mariadb-server
```

> Ejecutar `mysql_secure_installation`

```
sudo mysql_secure_installation
```

```
Enter current password for root (enter for none):

Switch to unix_socket authentication [Y/n] n

Change the root password? [Y/n] n

Remove anonymous users? [Y/n] y

Disallow root login remotely? [Y/n] y

Remove test database and access to it? [Y/n] y

Reload privilege tables now? [Y/n] y
```

Muestro sólo mis respuestas durante el script, el resto del output innecesario lo he borrado.

### Ejercicio 3
> Instalar un servidor de aplicaciones PHP-FPM

```
sudo apt install php-fpm
```


## VirtualHosting

### Ejercicio 1
> Crear VirtualHost con `server_name` `www.adrianjaramillo.tk`

Creo `/etc/nginx/sites-available/www.conf`:
```
server {
    listen 80;
    server_name www.adrianjaramillo.tk;

    location / {
        root   /var/www/www;
        index  index.html index.htm;
    }
}
```

Lo habilito:
```
sudo ln -s /etc/nginx/sites-available/www.conf /etc/nginx/sites-enabled/
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

> Crear nuevo registro CNAME en la zona DNS pública para `www.adrianjaramillo.tk`

![](https://i.imgur.com/RTvh7px.png)

### Ejercicio 2
> Redirige el VirtualHost por defecto `default` al VirtualHost de `www.adrianjaramillo.tk`

Añado la siguiente línea a `/etc/nginx/sites-available/default`:
```
return 301 $scheme://www.adrianjaramillo.tk$request_uri;
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Muestro que funciona la redirección:

![](https://i.postimg.cc/CxGv1xG4/nginx-redirect.gif)


## Mapeo de URL

### Ejercicio 1
> Redigir `www.adrianjaramillo.tk` a `www.adrianjaramillo.tk/principal`

Añado la siguiente línea a `/etc/nginx/sites-available/www.conf`:
```
rewrite ^/$ http://www.adrianjaramillo.tk/principal permanent;
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

En `principal`, creo un `index.html` de prueba.

Muestro que funciona la redirección:

![](https://i.postimg.cc/wTB81v9H/nginx-redireccion-principal.gif)

>En `principal` no se permite:
>
>- Listar ficheros
>- Enlaces simbólicos

Antes de nada, borro el `/var/www/www/principal/index.html` que previamente creé.

¿Razón? `ngx_http_autoindex_module` sólo empieza a funcionar **cuando no encuentra** un `index.html`.

Creo la siguiente estructura de prueba en `principal`:
```
blackmamba@kampe:/var/www/www/principal$ ls -la
total 16
drwxr-xr-x 2 root root 4096 Nov  9 22:23 .
drwxr-xr-x 3 root root 4096 Nov  9 19:55 ..
-rw-r--r-- 1 root root    9 Nov  9 19:55 fich1.txt
-rw-r--r-- 1 root root    9 Nov  9 19:56 fich2.txt
lrwxrwxrwx 1 root root    9 Nov  9 19:58 motd -> /etc/motd
```

Modifico `/etc/nginx/sites-available/www.conf`:
```
server {

    listen 80;
    server_name www.adrianjaramillo.tk;
    root /var/www/www;

    rewrite ^/$ http://www.adrianjaramillo.tk/principal permanent;

    location / {
        index index.html index.htm;
    }

    location /principal {
        autoindex off;
        disable_symlinks on;
    }

}
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Muestro que `autoindex off` funciona:

![](https://i.imgur.com/14d0P0e.png)

Intento acceder al enlace simbólico para mostrar que `disable_symlinks on` funciona:

![](https://i.imgur.com/T5DQJae.png)

### Ejercicio 2
> En `www.adrianjaramillo.tk/principal`, bajar una plantilla de `index.html` con tu nombre, y una lista de enlaces a las aplicaciones que desplegaremos

Me bajo la plantilla en `principal`:
```
sudo apt install unzip
sudo wget https://www.free-css.com/assets/files/free-css-templates/download/page270/univers.zip
sudo unzip univers.zip
```

Muestro que funciona y tiene mi nombre:

![](https://i.postimg.cc/dQ5VfySM/vps-web-principal.png)

### Ejercicio 3
> Al acceder a `www.adrianjaramillo.tk/principal/documentos` se visualizará `/srv/doc`    
>Se permitirá:
>
>- Listar ficheros
>- Enlaces simbólicos

Creo la siguiente estructura en `/srv/doc`:
```
blackmamba@kampe:/srv/doc$ ls -la
total 28
drwxr-xr-x 2 root root  4096 Nov 10 09:58 .
drwxr-xr-x 3 root root  4096 Nov 10 09:25 ..
-rw-r--r-- 1 root root 13264 Aug 27  2007 dummy.pdf
lrwxrwxrwx 1 root root    32 Nov 10 09:58 fich1.txt -> /var/www/www/principal/fich1.txt
-rw-r--r-- 1 root root  3028 Feb 24  2017 sample.pdf
```

Añado lo siguiente a `/etc/nginx/sites-available/www.conf`:
```
location /principal/documentos {
    alias /srv/doc;
    autoindex on;
    disable_symlinks off;
}
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Muestro que funciona `autoindex`:

![](https://i.imgur.com/5OXwFN6.png)

Muestro que se ven los ficheros:

![](https://i.imgur.com/3e6ZHe9.png)

Muestro que se ve el enlace simbólico:

![](https://i.imgur.com/hHv1s79.png)

### Ejercicio 4
> Redirigir errores 404 y 403 para todo el VirtualHost `www.adrianjaramillo.tk`. Crear 2 html dentro del directorio error.

Creo la siguiente estructura en `/error`:
```
blackmamba@kampe:/var/www/www/error$ ls -la
total 24
drwxr-xr-x 2 root root 4096 Nov 10 11:43 .
drwxr-xr-x 4 root root 4096 Nov 10 11:12 ..
-rw-r--r-- 1 root root 2518 Nov 10 11:43 403.html
-rw-r--r-- 1 root root 8788 Nov 10 11:22 404.html
```

Añado lo siguiente a `/etc/nginx/sites-available/www.conf`:
```
error_page 404 /error/404.html;
error_page 403 /error/403.html;  
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Muestro que el error 404 funciona accediendo a `/noexiste`:

![](https://i.imgur.com/3iS5wef.png)

Muestro que el error 403 funciona accediendo al enlace simbólico dentro de `/principal` *(recordamos que en ese directorio teníamos `disable_symlinks on`)*:

![](https://i.imgur.com/gpzUoCa.png)




## Autenticación

### Ejercicio 1
> Autenticación básica en `www.adrianjaramillo.tk/secreto`

Creo la siguiente estructura en `/var/www/www/secreto`:
```
blackmamba@kampe:/var/www/www/secreto$ ls -la
total 12
drwxr-xr-x 2 root root 4096 Nov 10 17:44 .
drwxr-xr-x 5 root root 4096 Nov 10 17:39 ..
-rw-r--r-- 1 root root  235 Nov 10 17:44 index.html
```

Instalo el paquete que nos proporciona el binario `htpasswd`, que necesitaremos:
```
sudo apt install apache2-utils
```

Creo el fichero de credenciales, añadiendo un usuario `admin` con contraseña `admin`:
```
sudo htpasswd -c /etc/nginx/.htpasswd admin
```

Añado lo siguiente a `/etc/nginx/sites-available/www.conf`:
```
location /secreto {
    auth_basic "Acceso restringido";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Muestro que funciona:

![](https://i.postimg.cc/qM7fW6JN/basic-auth-nginx.gif)




## PHP

### Ejercicio 1
> Comprobar la configuración de PHP-FPM por defecto *(socket unix o socket TCP)*

Para comprobarlo, ejecuto:
```
cat /etc/php/7.4/fpm/pool.d/www.conf | grep "listen ="
```
```
listen = /run/php/php7.4-fpm.sock
```
Vemos que por defecto PHP-FPM se configura escuchando en un socket unix.

> Configurar VirtualHost `www.adrianjaramillo.tk` para que pueda ejecutar PHP

Añado lo siguiente a `/etc/nginx/sites-available/www.conf`:
```
location ~ \.php$ {
    fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
    fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

### Ejercicio 2
> Crear un `info.php` que demuestre que funciona la ejecución PHP

En `/var/www/www/principal` me descargo un `info.php` de prueba:
```
sudo wget https://gist.githubusercontent.com/SyntaxC4/5648247/raw/94277156638f9c309f2e36e19bff378ba7364907/info.php
```

Muestro que funciona:

![](https://i.imgur.com/tSo7aRB.png)





## Ansible

Hacer configuración básica de Nginx con ansible, partiendo de: <https://fp.josedomingo.org/sri2122/u03/doc/ejercicio_proxy/ejercicio_proxy.zip>

Repositorio con mi configuración:  
<https://github.com/adriasir123/ansible-nginx>
