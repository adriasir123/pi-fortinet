---
title: "Tarea 8 Cloud:  Instalación de app python (Mezzanine)"
---

# Paso 1: instalación en desarrollo con entorno virtual
## Instalar python...etc
Instalamos todos los paquetes necesarios
```
sudo apt install python3 python3-venv python3-virtualenv virtualenv
```
## Creamos el entorno virtual
```
python3 -m venv mezzanine-pyenv
```
## Lo activamos
```
source mezzanine-pyenv/bin/activate
```
## Instalando Mezzanine
```
pip install mezzanine
```
## Creamos el proyecto
```
mezzanine-project my_mezzanine
```
Nos movemos dentro de ese directorio
```
cd my_mezzanine
```
## Creamos la base de datos
```
python manage.py createdb
```
> Durante la creación nos pide un usuario y contraseña. En mi caso, ha sido "Usuario: vagrant" "Contraseña: 1234".
> Pero, ¿qué es esta cuenta que hemos configurado?
> Pues es simplemente la cuenta que se usa a la hora de acceder a la zona de administración de la página web. 
> Es importante que no confundamos esta cuenta como si fuese la cuenta que estuviéramos creando para autenticarnos contra la base de datos, porque sqlite no tiene el concepto de usuarios y contraseñas para el acceso (usa los permisos del sistema) 


## Permitimos la conexión a la app web
En el fichero `local_settings.py`, tenemos que modificar la siguiente línea
```
ALLOWED_HOSTS = ["localhost", "127.0.0.1", "::1"]
```
Para quede de la siguiente manera
```
ALLOWED_HOSTS = ["localhost", "127.0.0.1", "::1", "172.22.0.245"]
```
Lo que estamos haciendo con esto, es permitir el acceso a la aplicación python, a la máquina donde nos queremos conectar (en mi caso es una máquina virtual vagrant)

## Ejecutamos el servidor web
```
python manage.py runserver 0.0.0.0:8000
```
> En mi caso lo abro permitiendo las conexiones desde cualquier IP, porque mi entorno de desarrollo es una VM vagrant

## Mostramos la web
![](https://i.imgur.com/x5DsdvA.png)
## Accediendo a la parte de administración
Para ello, tendremos que acceder a una URL parecida a la siguiente  
(digo parecida, porque simplemente lo que cambiaría sería la IP)
```
http://192.168.1.104:8000/admin/
```
Una vez dentro, la parte de administración sería tal que así
![](https://i.imgur.com/3JGcIZa.png)




# Paso 2: personalización de la web

## Cambiar nombre de la página
Dentro de la zona de administración, tendremos que irnos a "Site -> Settings -> Site -> Site Title". Una vez aquí, podremos cambiar el nombre de la web al que queramos:
![](https://i.imgur.com/CussH1y.png)
Cuando guardemos, y volvamos a la web principal, veremos que el nombre de la web se cambió correctamente
![](https://i.imgur.com/xDdXVKc.png)
## Añadir post con imagen
* Dentro de la zona de administración, tendremos que irnos a "Content -> Blog posts -> Add blog post". Una vez aquí, simplemente rellenaremos los campos necesarios, y crearemos el post.
* Cuando lo tengamos creado, nos aparecerá el nuevo post en la lista general de posts
![](https://i.imgur.com/2LI2L2u.png)* Para ver el post directamente en la web principal, la manera más sencilla y rápida sería hacer click en "View on site". Nuestro post ya terminado, quedaría tal que así:
![](https://i.imgur.com/N1DUceg.png)




# Paso 3: volcado de tablas y datos
Para ello, tendremos que ejecutar lo siguiente en el directorio donde tengamos la app web, y crearemos el fichero json con todos los datos que tenía la base de datos original SQLite
```
python manage.py dumpdata > db_mezzanine_bak.json
```
Luego de hacer esto, se nos quedaría el siguiente fichero json con los datos
```
-rw-r--r-- 1 vagrant vagrant  20750 Dec 17 09:29 db_mezzanine_bak.json
```


# Paso 4: guardado en Github

Por aquí dejo el [enlace](https://github.com/adriasir123/app_mezzanine) al repositorio donde están:
* Mi aplicación web Mezzanine creada
* La copia de seguridad de su base de datos (fichero json con el volcado que hicimos antes)

> He comentado el directorio `/static` en el fichero .gitignore de desarrollo, porque en ese directorio está guardada la imagen + thumbnails del post que creé en la web.  
Por lo tanto, es necesario que este directorio esté presente en producción


# Paso 5: despliegue en producción

Realiza el despliegue de la aplicación en tu entorno de producción (servidor web y servidor de base de datos en el cloud). Utiliza un entorno virtual. 

Como servidor de aplicación puedes usar gunicorn (crea una unidad systemd para gestionar este servicio)



## Instalando los paquetes que necesitaremos
```
sudo yum install python3 mariadb-devel.x86_64 gcc python3-devel
```
## Creamos el entorno virtual (en home)
```
python3 -m venv mezzanine-pyenv-prod
```
## Lo activamos
```
source mezzanine-pyenv-prod/bin/activate
```
## Instalando en el entorno virtual
```
pip install mezzanine gunicorn mysqlclient
```
> Después de hacer esto, en mi caso se me recomienda actualizar la versión de pip, ya que era muy antigua. Lo actualizo de la siguiente manera:
>```
>pip install --upgrade pip
>```

Después de instalar todo esto, podemos desactivar el entorno virtual
```
deactivate
```

## Descarga de Mezzanine desde repositorio
Una vez que ya tenemos lo necesario instalado en el entorno virtual, podremos pasar a descargarnos la app web de mezzanine que habíamos subido previamente a github.

* Nos movemos al directorio padre de donde queramos que esté localizado el document root de nuestra app web
```
cd /var/www/
```
* Una vez aquí, clonamos el repositorio
```
sudo git clone https://github.com/adriasir123/app_mezzanine.git
```
> En producción hacemos un clonado por https, ya que la idea es que en este repositorio en producción, simplemente hagamos pulls puntualmente.  
Así que nos interesa que git se configure por https en este caso en específico

* Por último en mi caso, quiero cambiar el nombre del document root, ya que cuando clonamos el repositorio, el directorio resultante lleva el nombre que tenía el repositorio
```
sudo mv app_mezzanine/ mezzanine
```

## Creación de la base de datos para Mezzanine en tortilla
* Nos logeamos como root
```
sudo mysql -u root -p
```
* Creamos la base de datos
```
CREATE DATABASE mezzanine CHARACTER SET utf8 COLLATE utf8_general_ci;
```
* Aplicamos todos los privilegios sobre todas las tablas que habrán en la base de datos para mezzanine al usuario mezzanineuser, tanto si accede a la base de datos desde localhost, como si accede desde remoto (cualquier IP de origen)
```
GRANT ALL ON mezzanine.* TO 'mezzanineuser'@'localhost' IDENTIFIED BY '1234';
```
```
GRANT ALL ON mezzanine.* TO 'mezzanineuser'@'%' IDENTIFIED BY '1234';
```
* Hacemos que se apliquen los privilegios
```
FLUSH PRIVILEGES;
```

## Carga de datos en tortilla
* Lo primero será modificar en salmorejo el fichero `settings.py` de nuestro Mezzanine en producción, para que contenga los datos correctos de conexión a la base de datos que acabamos de crear en tortilla

```
#############
# DATABASES #
#############

DATABASES = {
    "default": {
        # Add "postgresql_psycopg2", "mysql", "sqlite3" or "oracle".
        "ENGINE": "django.db.backends.mysql",
        # DB name or path to database file if using sqlite3.
        "NAME": "mezzanine",
        # Not used with sqlite3.
        "USER": "mezzanineuser",
        # Not used with sqlite3.
        "PASSWORD": "1234",
        # Set to empty string for localhost. Not used with sqlite3.
        "HOST": "10.0.0.14",
        # Set to empty string for default. Not used with sqlite3.
        "PORT": "3306",
    }
}
```
> Hacemos esto porque cuando vayamos a cargar los datos desde el fichero json, mirará en ese fichero para saber en qué base de datos tiene que cargar los datos

* Lo siguiente que tenemos que modificar en el `settings.py`, es el siguiente contenido sobre las keys 

```
#####################################
# KEYS ADDED FROM local_settings.py #
#####################################

SECRET_KEY = "oroll-r@3k)+ise#rhol*fa(z%(df9#52if_033vzys5*o_(80"
NEVERCACHE_KEY = "r9e-9_pro-iqwb4s)8ge4n$sq*$7b6bn=zy5=_v()*i(_q0d1#"
``` 

> Estas claves las hemos copiado del fichero `local_settings.py` que tenemos en la app en desarrollo.  
Los usos de la SECRET_KEY están descritos en el siguiente enlace https://stackoverflow.com/questions/15170637/effects-of-changing-djangos-secret-key/15383766#15383766  
Por ejemplo, uno de los usos que se menciona es el firmado de los objetos json.  
El fichero que nosotros tenemos para rellenar la base de datos para Mezzanine como lo teníamos en producción, es de formato json, y puede que en este fichero hubieran datos que utilizaran esta SECRET_KEY para funciones criptográficas...etc  
Además, con django debe de haber algún tipo de comprobación para mostrar un error en caso de que no se encuentren las claves necesarias


* Activamos el virtualenv (importante para que reconozca lo que voy a hacer a partir de ahora)
```
source ~/mezzanine-pyenv-prod/bin/activate
```

* Hacemos una creación inicial de las tablas
```
python manage.py migrate
```
> Esto toma en cuenta la definición de tablas que tuviera django en nuestra app en específico (o modelos de datos), no carga ningún tipo de datos. SÓLO CREA TABLAS VACÍAS

* Ejecutamos lo siguiente para cargar los datos del json (que contiene exactamente los mismos datos que la base de datos SQLite en desarrollo) en nuetra base de datos en tortilla para mezzanine
```
python manage.py loaddata db_mezzanine_bak.json
```
Si los datos se han conseguido cargar correctamente, el mensaje que recibiremos es el siguiente
```
Installed 151 object(s) from 1 fixture(s)
```





## Creación de la unidad systemd para gunicorn
* Abrimos un nuevo archivo para configurar la unidad
```
sudo nano /etc/systemd/system/gunicorn.service
```

* Le añadimos el siguiente contenido
```
[Unit]
Description = daemon gunicorn
After = network.target

[Service]
User = centos
Group = nginx
WorkingDirectory = /var/www/mezzanine
ExecStart = /home/centos/mezzanine-pyenv-prod/bin/gunicorn --access-logfile - --workers 3 --bind unix:/var/www/mezzanine/gunicorn.sock mezzanine.wsgi:application

[Install]
WantedBy = multi-user.target
```

* Recargamos la lista de servicios, para que el nuevo que hemos añadido, sea reconocido
```
sudo systemctl daemon-reload
```

* Ya podemos iniciar nuestro servicio
```
sudo systemctl start gunicorn
```

* Cambiar permisos del document root (porque en la unidad systemd los del grupo nginx son los que tienen permiso)
sudo chown -R nginx: /var/www/mezzanine/




## Configurando NGINX
* Creamos el fichero de configuración para el virtualhost de nginx
```
sudo touch /etc/nginx/conf.d/mezzanine.conf
```
* Le añadimos el siguiente contenido
```

```




## 




## Nuevo registro DNS para la app

python.aincrad-adrijara.gonzalonazareno.org



