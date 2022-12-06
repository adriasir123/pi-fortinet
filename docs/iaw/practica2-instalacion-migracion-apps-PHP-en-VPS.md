# Práctica 2: Instalación/migración de aplicaciones web PHP en tu VPS

## Tarea 1: Migración de phpBB

### Ejercicio 1
> Crear VirtualHost con `server_name` `portal.adrianjaramillo.tk`

Creo `/etc/nginx/sites-available/portal.conf`:
```
server {

    listen 80;
    server_name portal.adrianjaramillo.tk;
    root /var/www/portal;

    location / {
        index index.php index.html index.htm;
    }

    location ~ \.php$ {
        include        fastcgi_params;
        fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

}
```

Lo habilito:
```
sudo ln -s /etc/nginx/sites-available/portal.conf /etc/nginx/sites-enabled/
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Creo un registro CNAME en la zona DNS pública para `portal.adrianjaramillo.tk`:

![](https://i.postimg.cc/KcLLckzz/portal-tk-vps.png)

### Ejercicio 2
> Nombrar en `/etc/hosts` la BD como `bd.adrianjaramillo.tk`

Modifico la siguiente línea en `/etc/hosts`:
```
127.0.0.1       localhost.localdomain localhost bd.adrianjaramillo.tk
```

Muestro que funciona:
```
blackmamba@kampe:/etc/nginx/sites-available$ ping bd.adrianjaramillo.tk
PING localhost.localdomain (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost.localdomain (127.0.0.1): icmp_seq=1 ttl=64 time=0.039 ms
64 bytes from localhost.localdomain (127.0.0.1): icmp_seq=2 ttl=64 time=0.058 ms
^C
--- localhost.localdomain ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1031ms
rtt min/avg/max/mdev = 0.039/0.048/0.058/0.009 ms
```

### Ejercicio 3

Realizar la migración de phpBB.

#### Parte 1
> Instalar las dependencias de paquetería requeridas para que `phpBB` funcione

```
sudo apt install php-json php-mbstring php-xml php-mysql php-pdo php-zip php-gd php-curl php-pear php-bcmath
```

#### Parte 2
> Transferir el DocumentRoot de `phpBB` al VPS mediante repositorios

Hago una copia del DocumentRoot en el home:
```
sudo cp -r /var/www/phpBB ~
```

Cambio los propietarios:
```
sudo chown -R vagrant:vagrant ~/phpBB/
```

Inicializo el directorio como repo git:
```
git init
```

Creo el repositorio en github: <https://github.com/adriasir123/phpBB-document-root>

Subo el directorio al repositorio creado:
```
git remote add origin git@github.com:adriasir123/phpBB-document-root.git
git branch -M main
git add .
git config --global user.email "adristudy@gmail.com"
git config --global user.name "Adrián Jaramillo Rodríguez"
git commit -m "Initial commit"
git push -u origin main
```

Me bajo el repositorio en `/var/www/portal` del VPS:
```
sudo git clone https://github.com/adriasir123/phpBB-document-root.git .
```

Modifico los propietarios correctamente en `/var/www/portal/`:
```
sudo chown -R www-data:www-data /var/www/portal/
```

#### Parte 3
> Migrar la BD al VPS

Exporto la BD:
```
sudo mysqldump -u root phpBB_db > phpBB_db.sql
```

Creo la BD en el VPS:
```
CREATE DATABASE `phpBB_db`;
```

Creo un usuario con permisos sobre `phpBB_db`:

```
CREATE USER 'phpBB_user' IDENTIFIED BY '1234';

GRANT USAGE ON *.* TO 'phpBB_user'@localhost IDENTIFIED BY '1234';

GRANT ALL privileges ON `phpBB_db`.* TO 'phpBB_user'@localhost;

FLUSH PRIVILEGES;
```

Importo la copia de seguridad:
```
sudo mysql -u root phpBB_db < phpBB_db.sql
```

#### Parte 4
> Cambiar los datos de conexión a la base de datos

Modifico la siguiente línea en `config.php`:
```
$dbhost = 'bd.adrianjaramillo.tk';
```


### Ejercicio 4
> Comprobar que `phpBB` sigue funcionando en `portal.adrianjaramillo.tk`

![](https://i.postimg.cc/yx98q75x/phpbb-migracion.gif)





## Tarea 2: Instalación/migración Nextcloud

### Ejercicio 1

Instalar Nextcloud en el entorno de desarrollo.

#### Parte 1
> Crear el escenario

```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :nextcloudnginx do |nextcloudnginx|
    nextcloudnginx.vm.box = "debian/bullseye64"
    nextcloudnginx.vm.hostname = "nextcloudnginx"
  end

end
```

#### Parte 2
> Instalar y configurar Nginx

Lo instalo:
```
sudo apt update
sudo apt install curl gnupg2 ca-certificates lsb-release
sudo apt install nginx
```

Creo `/etc/nginx/sites-available/nextcloud.conf`, con `server_name` `www.nextcloud-adrianj.com`:
```
server {
    listen 80;
    server_name www.nextcloud-adrianj.com;

    # Path to the root of the domain
    root /var/www/nextcloud;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ^~ /.well-known {
        # The rules in this block are an adaptation of the rules
        # in the Nextcloud `.htaccess` that concern `/.well-known`.

        location = /.well-known/carddav { return 301 /cloud/remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /cloud/remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        # Let Nextcloud's API for `/.well-known` URIs handle all other
        # requests by passing them to the front-end controller.
        return 301 /cloud/index.php$request_uri;
    }

    location ^~ /cloud {
        # set max upload size and increase upload timeout:
        client_max_body_size 512M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;

        # Enable gzip but do not remove ETag headers
        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

        # Pagespeed is not supported by Nextcloud, so if your server is built
        # with the `ngx_pagespeed` module, uncomment this line to disable it.
        #pagespeed off;

        # HTTP response headers borrowed from Nextcloud `.htaccess`
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;

        # Remove X-Powered-By, which is an information leak
        fastcgi_hide_header X-Powered-By;

        # Specify how to handle directories -- specifying `/nextcloud/index.php$request_uri`
        # here as the fallback means that Nginx always exhibits the desired behaviour
        # when a client requests a path that corresponds to a directory that exists
        # on the server. In particular, if that directory contains an index.php file,
        # that file is correctly served; if it doesn't, then the request is passed to
        # the front-end controller. This consistent behaviour means that we don't need
        # to specify custom rules for certain paths (e.g. images and other assets,
        # `/updater`, `/ocm-provider`, `/ocs-provider`), and thus
        # `try_files $uri $uri/ /nextcloud/index.php$request_uri`
        # always provides the desired behaviour.
        index index.php index.html /cloud/index.php$request_uri;

        # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
        location = /cloud {
            if ( $http_user_agent ~ ^DavClnt ) {
                return 302 /cloud/remote.php/webdav/$is_args$args;
            }
        }

        # Rules borrowed from `.htaccess` to hide certain paths from clients
        location ~ ^/cloud/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)    { return 404; }
        location ~ ^/cloud/(?:\.|autotest|occ|issue|indie|db_|console)                  { return 404; }

        # Ensure this block, which passes PHP files to the PHP process, is above the blocks
        # which handle static assets (as seen below). If this block is not declared first,
        # then Nginx will encounter an infinite rewriting loop when it prepends
        # `/nextcloud/index.php` to the URI, resulting in a HTTP 500 error response.
        location ~ \.php(?:$|/) {
            # Required for legacy support
            rewrite ^/cloud/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /cloud/index.php$request_uri;

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;

            try_files $fastcgi_script_name =404;

            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_active true;     # Enable pretty urls
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;
        }

        location ~ \.(?:css|js|svg|gif|png|jpg|ico)$ {
            try_files $uri /cloud/index.php$request_uri;
            expires 6M;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }

        location ~ \.woff2?$ {
            try_files $uri /cloud/index.php$request_uri;
            expires 7d;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }

        # Rule borrowed from `.htaccess`
        location /cloud/remote {
            return 301 /cloud/remote.php$request_uri;
        }

        location /cloud {
            try_files $uri $uri/ /cloud/index.php$request_uri;
        }
    }
}
```
Nextcloud no está oficialmente soportado para Nginx, así que esta configuración está hecha por contribuidores.  
Igualmente, está publicada en la documentación oficial de Nextcloud: <https://docs.nextcloud.com/server/latest/admin_manual/installation/nginx.html#nextcloud-in-a-subdir-of-the-nginx-webroot>  
Específicamente, he elegido hacer la manera en subdirectorio, para que mi entorno de desarrollo y el de producción sean lo más parecidos posible. Así, la migración luego será más sencilla.


Lo habilito:
```
sudo ln -s /etc/nginx/sites-available/nextcloud.conf /etc/nginx/sites-enabled/
```

Compruebo que la configuración sea correcta:
```
sudo nginx -t
```
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Modifico mi `/etc/hosts`:
```
# nextcloudnginx resolutions
192.168.121.62 www.nextcloud-adrianj.com
```

#### Parte 3
> Instalar y configurar php

Instalo PHP-FPM y dependencias php para Nextcloud:
```
sudo apt install php-fpm php-mysql php-zip php-xml php-mbstring php-gd php-curl php-cli php-common php-json php-intl php-pear php-imagick php-dev php-soap php-bz2
```

Modifico las siguientes líneas en `/etc/php/7.4/fpm/php.ini` y `/etc/php/7.4/cli/php.ini`:
```
date.timezone = Europe/Madrid
cgi.fix_pathinfo=0
```

Descomentar las siguientes líneas en `/etc/php/7.4/fpm/pool.d/www.conf`:
```
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

Habilito y reinicio php-fpm:
```
sudo systemctl enable php7.4-fpm
sudo systemctl restart php7.4-fpm
```

#### Parte 4
> Instalar servidor MariaDB

```
sudo apt install mariadb-server
```

Creo la BD para Nextcloud:
```
CREATE DATABASE `nextcloud_db`;
```

Creo un usuario con permisos sobre `nextcloud_db`:
```
CREATE USER 'nextcloud_user' IDENTIFIED BY '1234';

GRANT USAGE ON *.* TO 'nextcloud_user'@localhost IDENTIFIED BY '1234';

GRANT ALL privileges ON `nextcloud_db`.* TO 'nextcloud_user'@localhost;

FLUSH PRIVILEGES;
```

#### Parte 5
> Descargar Nextcloud

En `/var/www/nextcloud` hago lo siguiente:
```
sudo apt install unzip
sudo wget https://download.nextcloud.com/server/releases/nextcloud-22.2.1.zip
sudo unzip nextcloud-22.2.1.zip
sudo mv nextcloud cloud
```

Cambio los propietarios:
```
sudo chown -R www-data:www-data /var/www/nextcloud/
```

Así, consigo tener Nextcloud descargado en un directorio `cloud` dentro del Document Root `/var/www/nextcloud`, lo cual será una situación muy parecida a lo que tendré en el entorno de producción.

#### Parte 6
> Instalación de Nextcloud

Para empezar, accedo a `www.nextcloud-adrianj.com/cloud`

Datos de la cuenta admin:

- Username: admin
- Password: admin1

Introduciendo esos datos, junto con los de conexión a la BD, instalamos:

![](https://i.imgur.com/WqGri9t.png)

Después de esto, ya tendríamos Nextcloud funcionando:

![](https://i.imgur.com/5rkiVm8.png)



### Ejercicio 2

Realizar la migración de Nextcloud, para que se acceda en `www.adrianjaramillo.tk/cloud`.

#### Parte 1
> Instalar dependencias y configurar php

Instalo los siguientes paquetes necesarios para que Nextcloud funcione, y que faltan en el VPS:
```
sudo apt install php-cli php-intl php-imagick php-dev php-soap php-bz2
```

Modifico las siguientes líneas en `/etc/php/7.4/fpm/php.ini` y `/etc/php/7.4/cli/php.ini`:
```
date.timezone = Europe/Madrid
cgi.fix_pathinfo=0
```

Descomentar las siguientes líneas en `/etc/php/7.4/fpm/pool.d/www.conf`:
```
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

Habilito y reinicio php-fpm:
```
sudo systemctl enable php7.4-fpm
sudo systemctl restart php7.4-fpm
```

#### Parte 2
> Transferir el subdirectorio de `Nextcloud` al VPS mediante repositorios

Hago una copia del subdirectorio que contiene la app en el home:
```
sudo cp -r /var/www/nextcloud/cloud ~
```

Cambio los propietarios:
```
sudo chown -R vagrant:vagrant ~/cloud/
```

Inicializo el directorio como repo git:
```
git init
```

Creo el repositorio en github: <https://github.com/adriasir123/nextcloud-subdirectorio-cloud>

Subo el directorio al repositorio creado:
```
git remote add origin git@github.com:adriasir123/nextcloud-subdirectorio-cloud.git
git branch -M main
git add .
git config --global user.email "adristudy@gmail.com"
git config --global user.name "Adrián Jaramillo Rodríguez"
git commit -m "Initial commit"
git push -u origin main
```

Me bajo el repositorio en `/var/www/www/cloud` del VPS:
```
sudo git clone https://github.com/adriasir123/nextcloud-subdirectorio-cloud.git .
```

Modifico los propietarios correctamente en `/var/www/www/cloud/`:
```
sudo chown -R www-data:www-data /var/www/www/cloud/
```

#### Parte 3
> Migrar la BD al VPS

Exporto la BD:
```
sudo mysqldump -u root nextcloud_db > nextcloud_db.sql
```

Creo la BD para Nextcloud:
```
CREATE DATABASE `nextcloud_db`;
```

Creo un usuario con permisos sobre `nextcloud_db`:
```
CREATE USER 'nextcloud_user' IDENTIFIED BY '1234';

GRANT USAGE ON *.* TO 'nextcloud_user'@localhost IDENTIFIED BY '1234';

GRANT ALL privileges ON `nextcloud_db`.* TO 'nextcloud_user'@localhost;

FLUSH PRIVILEGES;
```

Importo la copia de seguridad:
```
sudo mysql -u root nextcloud_db < nextcloud_db.sql
```

#### Parte 4
> Cambiar el fichero de configuración de Nextcloud

Modifico `/var/www/www/cloud/config/config.php`:
```
<?php
$CONFIG = array (
  'instanceid' => 'ocr2lp6lisyt',
  'passwordsalt' => 'RjrrSdcDyvgGXhx/Grnoccv298gljL',
  'secret' => 'b7JkIA6v1+MGT70voUpA9XOvvMRVAc/PsgImxoDmYQytgHut',
  'trusted_domains' =>
  array (
    0 => 'www.adrianjaramillo.tk',
  ),
  'datadirectory' => '/var/www/www/cloud/data',
  'dbtype' => 'mysql',
  'version' => '22.2.1.2',
  'overwrite.cli.url' => 'http://www.adrianjaramillo.tk/cloud',
  'dbname' => 'nextcloud_db',
  'dbhost' => 'bd.adrianjaramillo.tk',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextcloud_user',
  'dbpassword' => '1234',
  'installed' => true,
);
```

#### Parte 5
> Configurar Nginx

Hago una copia de seguridad del VirtualHost que voy a modificar:
```
sudo cp www.conf www.conf.backup
```

Modifico `www.conf`:
```
server {

    ###########
    # GENERAL #
    ###########

    listen 80;
    server_name www.adrianjaramillo.tk;

    root /var/www/www;
    index index.html index.htm;

    error_page 404 /error/404.html;
    error_page 403 /error/403.html;

    #############
    # PRINCIPAL #
    #############

    rewrite ^/$ http://www.adrianjaramillo.tk/principal permanent;

    location /principal {
        autoindex off;
        disable_symlinks on;
    }

    location /principal/documentos {
        alias /srv/doc;
        autoindex on;
        disable_symlinks off;
    }

    location /secreto {
        auth_basic "Acceso restringido";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }

    #############
    # NEXTCLOUD #
    #############

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ^~ /.well-known {
        # The rules in this block are an adaptation of the rules
        # in the Nextcloud `.htaccess` that concern `/.well-known`.

        location = /.well-known/carddav { return 301 /cloud/remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /cloud/remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        # Let Nextcloud's API for `/.well-known` URIs handle all other
        # requests by passing them to the front-end controller.
        return 301 /cloud/index.php$request_uri;
    }

    location ^~ /cloud {
        # set max upload size and increase upload timeout:
        client_max_body_size 512M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;

        # Enable gzip but do not remove ETag headers
        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

        # Pagespeed is not supported by Nextcloud, so if your server is built
        # with the `ngx_pagespeed` module, uncomment this line to disable it.
        #pagespeed off;

        # HTTP response headers borrowed from Nextcloud `.htaccess`
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;

        # Remove X-Powered-By, which is an information leak
        fastcgi_hide_header X-Powered-By;

        # Specify how to handle directories -- specifying `/nextcloud/index.php$request_uri`
        # here as the fallback means that Nginx always exhibits the desired behaviour
        # when a client requests a path that corresponds to a directory that exists
        # on the server. In particular, if that directory contains an index.php file,
        # that file is correctly served; if it doesn't, then the request is passed to
        # the front-end controller. This consistent behaviour means that we don't need
        # to specify custom rules for certain paths (e.g. images and other assets,
        # `/updater`, `/ocm-provider`, `/ocs-provider`), and thus
        # `try_files $uri $uri/ /nextcloud/index.php$request_uri`
        # always provides the desired behaviour.
        index index.php index.html /cloud/index.php$request_uri;

        # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
        location = /cloud {
            if ( $http_user_agent ~ ^DavClnt ) {
                return 302 /cloud/remote.php/webdav/$is_args$args;
            }
        }

        # Rules borrowed from `.htaccess` to hide certain paths from clients
        location ~ ^/cloud/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)    { return 404; }
        location ~ ^/cloud/(?:\.|autotest|occ|issue|indie|db_|console)                  { return 404; }

        # Ensure this block, which passes PHP files to the PHP process, is above the blocks
        # which handle static assets (as seen below). If this block is not declared first,
        # then Nginx will encounter an infinite rewriting loop when it prepends
        # `/nextcloud/index.php` to the URI, resulting in a HTTP 500 error response.
        location ~ \.php(?:$|/) {
            # Required for legacy support
            rewrite ^/cloud/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /cloud/index.php$request_uri;

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;

            try_files $fastcgi_script_name =404;

            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_active true;     # Enable pretty urls
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;
        }

        location ~ \.(?:css|js|svg|gif|png|jpg|ico)$ {
            try_files $uri /cloud/index.php$request_uri;
            expires 6M;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }

        location ~ \.woff2?$ {
            try_files $uri /cloud/index.php$request_uri;
            expires 7d;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }

        # Rule borrowed from `.htaccess`
        location /cloud/remote {
            return 301 /cloud/remote.php$request_uri;
        }

        location /cloud {
            try_files $uri $uri/ /cloud/index.php$request_uri;
        }
    }
}
```

Después de esto, reinicio tanto Nginx como PHP-FPM para asegurarme de que no tendré problemas al hacer las pruebas:
```
sudo systemctl restart nginx.service
sudo systemctl restart php7.4-fpm.service
```

#### Parte 6
> Comprobar que Nextcloud funciona en el VPS accediendo a `www.adrianjaramillo.tk/cloud`

![](https://i.postimg.cc/J4sKCRW2/nextcloud-vps.gif)

#### Parte 7
> Comprobar que la página principal sigue funcionando, junto con las configuraciones que tenía

![](https://i.postimg.cc/Yq2vjBff/principal-sigue-funcionando-tras-migracion-nextcloud.gif)



### Ejercicio 3
> Instalar en tu máquina el cliente de Nextcloud y configurarlo para el acceso al VPS.

Está disponible en los repositorios de Debian, por lo que:
```
sudo apt update
sudo apt install nextcloud-desktop
```

Paso a configurarlo:

![](https://i.imgur.com/LNwDmov.png)

![](https://i.imgur.com/lqI9BUp.png)

Nos pide autenticarnos desde el navegador antes de entrar:

![](https://i.imgur.com/IBEgsEr.png)

El directorio local indicado, es el que se utilizará para la sincronización:

![](https://i.imgur.com/6Vxpb3j.png)

Así se vería nuestro cliente después de haber sincronizado y estar funcionando:

![](https://i.imgur.com/7A6xIRM.png)

Abajo muestro el directorio local que me ha sincronizado el cliente:

![](https://i.imgur.com/f6ecjdT.png)



## Tarea 3
> Modificar la web principal con accesos a las 2 apps migradas

![](https://i.postimg.cc/8CJshYG0/principal-accesos-apps.gif)
