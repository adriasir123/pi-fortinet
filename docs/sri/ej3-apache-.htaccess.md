# Ejercicio 3: Configuración de Apache mediante .htaccess

## ENTREGA

### Parte 1
> Captura del acceso a `ej3-apache-htaccess.x10host.com/nas`

![](https://i.imgur.com/L1G3jir.png)

### Parte 2
> Entregar captura donde se muestre la redirección de `ej3-apache-htaccess.x10host.com/google` a `www.google.es`

![](https://i.imgur.com/QzmVPzj.png)

### Parte 3
> Mostrar que funciona la autenticación en `ej3-apache-htaccess.x10host.com/prohibido`

![](https://i.postimg.cc/j2HHLF6K/acceso-prohibido-apache.gif)







## EJERCICIOS

### Ejercicio 0
> Si necesitamos configurar el servidor web del hosting, ¿qué podemos hacer?

Usar ficheros `.htaccess` donde los necesitemos *(se pueden usar varios en un mismo VirtualHost, pero siempre 1 sólo por directorio)*.

> ¿Para qué sirve la directiva AllowOverride de Apache?

Permite el uso de ficheros `.htaccess` para sobreescribir la configuración de Apache a partir de un directorio específico.

### Ejercicio 1
> Habilitar Indexes en la URL `ej3-apache-htaccess.x10host.com/nas`

Creo el fichero `/domains/ej3-apache-htaccess.x10host.com/public_html/nas/.htaccess` con el contenido:
```
Options +Indexes
```

Muestro que funciona:

![](https://i.imgur.com/L1G3jir.png)

### Ejercicio 2
> Crear una redirección permanente para `ej3-apache-htaccess.x10host.com/google` a `www.google.es`

Creo el fichero `/domains/ej3-apache-htaccess.x10host.com/public_html/.htaccess` con el contenido:
```
RedirectPermanent /google https://www.google.es/
```

Muestro que funciona:

![](https://i.postimg.cc/DyZ7zTyv/redirect-permanent-google.gif)

### Ejercicio 3
> Pedir autenticación en `ej3-apache-htaccess.x10host.com/prohibido`

Creo el fichero `/domains/ej3-apache-htaccess.x10host.com/public_html/prohibido/.htpasswd`:
```
admin:$apr1$qh75ojvl$AhjZXOFVsd.5.zRlmVGDj0
```

El contenido está generado con <https://hostingcanada.org/htpasswd-generator/>.  

Importante usar el modo _"Apache specific salted MD5"_, los más avanzados no funcionan con x10hosting.

Las credenciales de acceso que he creado son:

- Username: admin
- Password: admin

Creo el fichero `/domains/ej3-apache-htaccess.x10host.com/public_html/prohibido/.htaccess` con el contenido:
```
AuthType Basic
AuthName "Access restricted"
AuthUserFile .htpasswd
Require valid-user
```

Muestro que funciona:

![](https://i.postimg.cc/j2HHLF6K/acceso-prohibido-apache.gif)
