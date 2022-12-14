---
title: "Práctica: Despliegue de CMS java"
---

OUTLINE para hacer:

Conexiones a tener en guacamole:
-1 VNC linux
-1 VNC windows
-SSH


Funcionalidad añadida de grabaciones
-Grabaciones en vídeo de las sesión ssh (se graba por sesión)


Funcionalidades que no he podido añadir
-RDP
freerdp2-dev (la que se puede instalar en buster, pero el configure no detecta esa librería para que funcione RDP)
libfreerdp-dev (la que te pide el manual de guacamole, está en stretch, pero fue eliminada de buster por BUGS...etc)

-Telnet (por errores en Windows7 no se activaba correctamente telnet)





# 1. Indica la aplicación escogida y su funcionalidad.

Apache Guacamole

Consta de un cliente (app web) y un servidor (guacd, que establece las conexiones remotas)




# 2. Escribe una guía de los pasos fundamentales para realizar la instalación

## Requisitos previos

-Lo necesitamos para poder compilar guacamole server
sudo apt install build-essential

install java
https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-debian-10

sudo apt install default-jre
java -version


install tomcat9
https://tecadmin.net/install-apache-tomcat-9-on-debian/

sudo apt install tomcat9

## Instalación


## Configuración guacamole
GUACAMOLE_HOME is the name given to Guacamole's configuration directory, which is located at /etc/guacamole by default. All configuration files, extensions, etc. reside within this directory. The structure of GUACAMOLE_HOME is rigorously defined

BUT

If you cannot or do not wish to use /etc/guacamole for GUACAMOLE_HOME, the location can be overridden through any of the following methods:
Creating a directory named .guacamole, within the home directory of the user running the servlet container. This directory will automatically be used for GUACAMOLE_HOME if it exists.


Por esto, tendremos que crear el directorio
```
sudo mkdir /.guacamole
```








UTILIZAR EL COMANDO PATCH, COGER EL PARCHE, LLEVARLO A UN FICHERO, Y APLICÁRSELO AL FICHERO C CORRESPONDIENTE.
LA SINTAXIS NO ERA DE GIT, ERA UN DIFF FILE, UN FICHERO DE DIFERENCIAS

patch guacenc.c ~/patch
(comando usado para evitar el error que tenía al compilar, teniendo las librerías necesarias para la grabación de sesiones)

Enlace al parche por el error al compilar con la funcionalidad de grabación (librerías de vídeo instaladas)
https://issues.apache.org/jira/browse/GUACAMOLE-638














# 3. ¿Has necesitado instalar alguna librería? ¿Has necesitado instalar un conector de una base de datos?


# 4. Entrega una captura de pantalla donde se vea la aplicación funcionando.



# 5. Realiza la configuración necesaria en apache2 y tomcat (utilizando el protocolo AJP) para que la aplicación sea servida por el servidor web.


# 6. Entrega una captura de pantalla donde se vea la aplicación funcionando servida por apache2.






Working guides followed

install guacamole
https://guacamole.apache.org/doc/gug/installing-guacamole.html#deploying-guacamole

configuring guacamole
https://guacamole.apache.org/doc/gug/configuring-guacamole.html#guacamole-home






Guides not followed, maybe working

guacamole debian guide (not followed, not sure if works)
https://wiki.debian.org/Guacamole

tecmint guide for guacamole (not followed)
https://www.tecmint.com/guacamole-access-remote-linux-windows-machines-via-web-browser/






[ Important data ]

- Password vnc server connections (linux server and Windows)
passsword 12345678 (NOT VIEW ONLY)

Longitud máxima de 8 caracteres en las contraseñas

- Password for vagrant user (porque pasé el paquete de guacamole server y client por scp y me logueé con contraseña)
1234

- Administrative password TightVNC Server (en windows)
12345678