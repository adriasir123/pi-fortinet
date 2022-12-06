# Práctica: Despliegue de CMS java

## Parte 1

> Indicar la aplicación escogida y su funcionalidad

Elijo OpenCms.

OpenCms está enfocado a crear páginas web en entornos profesionales tanto pequeños como grandes, en intranets y en Internet. Sus funcionalidades a destacar son:

* Interfaz web avanzada para estructurar el sitio
* Creación de contenido con un editor WYSIWYG
* Webs multilenguaje editables por múltiples usuarios con diferentes permisos
* Posibilidad de añadir nuevas funcionalidades creando tus propios módulos
* Generación de PDF
* Formularios web

## Parte 2

> Escribir una guía con los pasos fundamentales para realizar la instalación

Instalo Tomcat:

```console
sudo apt install tomcat9
```

Compruebo que funciona:

![tomcatfuncionandobase](https://i.imgur.com/yDA3Hc2.png)

Instalo mariadb:

```console
sudo apt install mariadb-server
```

Añado una contraseña al usuario root:

```console
ALTER USER 'root'@'localhost' IDENTIFIED BY 'vagrant';
```

En `/tmp` descargo y extraigo el zip de opencms:

```console
wget http://www.opencms.org/downloads/opencms/opencms-13.0.zip
unzip opencms-13.0.zip
```

Muevo `opencms.war` a `/var/lib/tomcat9/webapps` para que se despliegue:

```console
sudo mv opencms.war /var/lib/tomcat9/webapps
```

Inicio la instalación del CMS:

![instalacion1](https://i.imgur.com/G39QGS2.png)

![instalacion2](https://i.imgur.com/QG2HypV.png)

![instalacion3](https://i.imgur.com/jWNzDlk.png)

(la contraseña es la que antes añadimos, vagrant)

![instalacion4](https://i.imgur.com/cc0DtJr.png)

![instalacion5](https://i.imgur.com/8gefccc.png)

![instalacion6](https://i.imgur.com/oQ6i0bE.png)

![instalacion7](https://i.imgur.com/1RfXNqh.png)

## Parte 3

> ¿Has necesitado instalar alguna librería? ¿Has necesitado instalar un conector de una base de datos?

No a las 2 preguntas, no se han tenido que instalar elementos adicionales.

Teniendo Tomcat, MariaDB, y haciendo la instalación inicial del CMS en la web, no se necesita nada más dada la simpleza de OpenCms.

## Parte 4

> Entregar una captura de pantalla donde se vea OpenCms funcionando desde Tomcat

![opencmstomcat](https://i.imgur.com/A9SqG98.jpg)

## Parte 5

> Realizar la configuración necesaria en Nginx para que OpenCms sea servido por el servidor web

Instalo Nginx:

```console
sudo apt install nginx
```

Creo `/etc/nginx/sites-available/opencms.conf`:

```console
server {

    listen 80;
    server_name www.opencms.com;

    location / {
        proxy_pass http://localhost:8080;
        include proxy_params;
    }

}
```

Lo habilito:

```console
sudo ln -s /etc/nginx/sites-available/opencms.conf /etc/nginx/sites-enabled
```

Reinicio Nginx:

```console
sudo systemctl restart nginx
```

Añado lo siguiente a mi `/etc/hosts`:

```console
# CMS Java
192.168.121.89 www.opencms.com
```

Pruebo que funciona:

![opencmsnginx](https://i.imgur.com/nw2kQwb.jpg)
