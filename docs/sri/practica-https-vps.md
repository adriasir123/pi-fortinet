# Práctica: Configuración de HTTPS en el VPS

Se configurará para todos los VirtualHosts usando [Let's Encrypt
](https://letsencrypt.org).



## Parte 1
> Comprobar que Brave tiene el certificado de Let’s Encrypt

Antes de comprobar nada, es esencial entender su chain of trust:

![](https://letsencrypt.org/images/isrg-hierarchy.png)

Como vemos, existen muchas entidades certificadoras, pero las que nos interesan son `ISRG Root X1` y `R3`. Ésta última es la que suele firmar a los usuarios finales.

Brave no tiene el certificado de `R3`, sino el de `ISRG Root X1`:

![](https://i.imgur.com/0BqmthE.png)

¿Por qué esto es así? Por la chain of trust que he mostrado arriba.

Podemos hacer una prueba de esto, si accedemos a la web oficial de Let’s Encrypt y mostramos la jerarquía del certificado:

![](https://i.imgur.com/LqYHkrH.png)

Directamente Brave "no confía" en `R3`, porque no tiene su certificado y por lo cual no puede verificar su firma, pero sí tiene el certificado root de `ISRG Root X1` que firmó a R3, entonces puede verificar esa firma en el certificado de `R3` y en consecuencia, confiar en él.



## Parte 2

### Preguntas

**¿Qué función tiene el cliente ACME?**

Verificar que somos dueños del nombre de dominio, y emitirnos un certificado.

[Hay muchos clientes](https://letsencrypt.org/docs/client-options/), pero el recomendado es Certbot.

**¿Qué configuración se realiza en el servidor web?**

1. Solicitamos el certificado
2. Configuramos los VirtualHosts con el certificado

**¿Qué pruebas realiza Let’s Encrypt para asegurar que somos los administrados del sitio web?**

Let's Encrypt realiza "challenges", y hay [varios tipos](https://letsencrypt.org/docs/challenge-types/). Los más comunes son HTTP-01 y DNS-01.

En mi caso, como querré obtener un certificado wildcard, tendré que hacer el DNS-01.

Su funcionamiento es simple:

1. Let’s Encrypt nos proporciona un registro TXT que tendremos que crear en nuestro DNS bajo el nombre `_acme-challenge.adrianjaramillo.tk.`
2. Let’s Encrypt hace una consulta a `_acme-challenge.adrianjaramillo.tk.`
3. Si la consulta resolvió el nombre correctamente con la cadena que se nos proporcionó, Let’s Encrypt verifica que somos los dueños del dominio, y nos proporciona el certificado firmado.

**¿Se puede usar el DNS para verificar que somos administradores del sitio?**

Sí. De hecho, es el único challenge permitido si queremos obtener un certificado wildcard.

### Certificado wildcard `*.adrianjaramillo.tk`

Instalo certbot y paquetes relacionados:
```
sudo apt install letsencrypt
```

Añado el siguiente registro a mi DNS:

![](https://i.imgur.com/84y17Zw.png)

Pido el certificado:
```
sudo certbot certonly --manual --preferred-challenges dns -d *.adrianjaramillo.tk
```
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Requesting a certificate for *.adrianjaramillo.tk
Performing the following challenges:
dns-01 challenge for adrianjaramillo.tk

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.adrianjaramillo.tk with the following value:

Zjtt4Rws9cuSpcQ8yMGJ5cXGcMfhgWUPen36G2WOwSg

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
   Your certificate will expire on 2022-03-06. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Verifico que tengo el certificado:
```
sudo certbot certificates
```
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: adrianjaramillo.tk
    Serial Number: 348d745d49f1968da5ccfce5d50d4b3018d
    Key Type: RSA
    Domains: *.adrianjaramillo.tk
    Expiry Date: 2022-03-06 18:09:13+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/adrianjaramillo.tk/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/adrianjaramillo.tk/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```



## Parte 3
> Configurar 2 ficheros por VirtualHost (HTTP y HTTPS). Redirigir HTTP a HTTPS.

`django_http.conf`:
```
server {

    listen 80;
    server_name django.adrianjaramillo.tk;

    return 301 https://$host$request_uri;

}
```

`django_https.conf`:
```
server {

    listen 443 ssl;
    server_name django.adrianjaramillo.tk;
    root /home/blackmamba/django_tutorial;

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
        proxy_pass http://localhost:8080;
        include proxy_params;
    }

    location /static/polls {
        alias /home/blackmamba/django_tutorial/polls/static/polls;
    }

    location /static/admin {
        alias /home/blackmamba/venv/lib/python3.9/site-packages/django/contrib/admin/static/admin/;
    }

}
```

`portal_http.conf`:
```
server {

    listen 80;
    server_name portal.adrianjaramillo.tk;

    return 301 https://$host$request_uri;

}
```

`portal_https.conf`:
```
server {

    listen 443 ssl;
    server_name portal.adrianjaramillo.tk;
    root /var/www/portal;

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
        index index.php index.html index.htm;
    }

    location ~ \.php$ {
        include        fastcgi_params;
        fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

}
```

`www_http.conf`:
```
server {

    listen 80;
    server_name www.adrianjaramillo.tk;

    return 301 https://$host$request_uri;

}
```

`www_https.conf`:
```
server {

    ###########
    # GENERAL #
    ###########

    listen 443 ssl;
    server_name www.adrianjaramillo.tk;

    root /var/www/www;
    index index.html index.htm;

    error_page 404 /error/404.html;
    error_page 403 /error/403.html;

    location ~ \.php$ {
        fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    ###################
    # RSA certificate #
    ###################

    ssl_certificate /etc/letsencrypt/live/adrianjaramillo.tk/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/adrianjaramillo.tk/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;

    #############
    # PRINCIPAL #
    #############

    rewrite ^/$ https://www.adrianjaramillo.tk/principal permanent;

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



## Parte 4
> Comprobar que se ha creado una tarea cron para renovar el certificado

`/etc/cron.d/certbot`:
```
# /etc/cron.d/certbot: crontab entries for the certbot package
#
# Upstream recommends attempting renewal twice a day
#
# Eventually, this will be an opportunity to validate certificates
# haven't been revoked, etc.  Renewal will only occur if expiration
# is within 30 days.
#
# Important Note!  This cronjob will NOT be executed if you are
# running systemd as your init system.  If you are running systemd,
# the cronjob.timer function takes precedence over this cronjob.  For
# more details, see the systemd.timer manpage, or use systemctl show
# certbot.timer.
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew
```

La renovación *sólo ocurrirá* si la expiración es en 30 días.

Podemos usar herramientas online para entender la expresión cron:

![](https://i.imgur.com/rOZ8VZI.png)

Es decir, la tarea cron se ejecuta cada 12 horas.



## Parte 5
> Comprobar que las páginas son accesibles por HTTPS y visualizar los detalles del certificado

En todos los VirtualHosts, los detalles del certificado son los mismos:

![](https://i.imgur.com/JVToH7q.png)

Muestro el resto de VirtualHosts:

![](https://i.imgur.com/Ml1Vk9O.png)

![](https://i.imgur.com/46Er7Cv.png)

![](https://i.imgur.com/5gTprLu.png)



## Parte 6
> Modificar el cliente Nextcloud para que siga funcionando con HTTPS

Desde que hemos configurado HTTPS, el cliente da errores:

![](https://i.imgur.com/qHOuuHo.png)

Tenemos que borrar la conexión antigua, y crear una nueva HTTPS:

![](https://i.imgur.com/xbdnz3r.png)

Una vez terminada la configuración, vemos que sigue funcionando con HTTPS:

![](https://i.imgur.com/DfZe1vx.png)

Además, si queremos, el cliente de Nextcloud nos permite ver los detalles del certificado:

![](https://i.postimg.cc/L6H3fLHH/nextcloud-client-https.gif)
