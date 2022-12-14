---
title: "Ejercicio 4: Mapear URLs a ubicaciones de un sistema de ficheros"
---

# EJ 0

## Crea un nuevo host virtual que es accedido con el nombre www.mapeo.com, cuyo DocumentRoot sea /srv/mapeo

En `/etc/hosts` del cliente...
```
##########################################
# EJ4 Mapeo URL resolutions
##########################################
172.22.5.252	www.mapeo.com
##########################################
```

En `/etc/apache2/sites-available/mapeo.conf`...
```
<VirtualHost *:80>

        ServerName www.mapeo.com
        DocumentRoot /srv/mapeo

        <Directory /srv/>
            Options Indexes FollowSymLinks
            AllowOverride None
            Require all granted
        </Directory>


        # ErrorLog ${APACHE_LOG_DIR}/error.log
        # CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

Creamos el document root
```
sudo mkdir /srv/mapeo
```

Habilitamos el site para que esté disponible
```
sudo a2ensite mapeo.conf
```

Reiniciamos Apache
```
sudo systemctl reload apache2
```

Creamos una página de test para comprobar que funcione
![](https://i.imgur.com/JMpyvGe.png)










# EJ 1

## Cuando se entre a la dirección www.mapeo.com se redireccionará automáticamente a www.mapeo.com/principal, donde se mostrará el mensaje de bienvenida.


































# EJ 2

## En el directorio principal no se permite ver la lista de los ficheros, no se permite que se siga los enlaces simbólicos y no se permite negociación de contenido. Muestra al profesor el funcionamiento. ¿Qué configuración tienes que poner?



# EJ 3

## Si accedes a la página www.mapeo.com/principal/documentos se visualizarán los documentos que hay en /home/usuario/doc. Por lo tanto se permitirá el listado de fichero y el seguimiento de enlaces simbólicos siempre que el propietario del enlace y del fichero al que apunta sean el mismo usuario. Explica bien y pon una prueba de funcionamiento donde se vea bien el seguimiento de los enlaces simbólicos.






# EJ 4 

## En todo el host virtual se debe redefinir los mensajes de error de objeto no encontrado y no permitido. Para el ello se crearan dos ficheros html dentro del directorio error. Entrega las modificaciones necesarias en la configuración y una comprobación del buen funcionamiento.



