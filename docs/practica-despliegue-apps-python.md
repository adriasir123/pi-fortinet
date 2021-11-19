# Práctica: Despliegue de aplicaciones python

## Tarea 1: Entorno de desarrollo

tutorial de django 3.2 (https://docs.djangoproject.com/en/3.2/intro/tutorial01/)

### Parte 0
> Crear escenario

```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :desplieguepython do |desplieguepython|
    desplieguepython.vm.box = "debian/bullseye64"
    desplieguepython.vm.hostname = "desplieguepython"
  end

end
```

### Parte 1
> Realizar fork del repositorio: https://github.com/josedom24/django_tutorial

![](https://i.imgur.com/eqNYbrr.png)

> Clonar el repo en el entorno de desarrollo

```
git clone git@github.com:adriasir123/django_tutorial.git
```

### Parte 2
> Crear un entorno virtual

```
mkdir venv
cd venv
sudo apt update
sudo apt install python3-venv
python3 -m venv .
source ~/venv/bin/activate
```

> Instalar dependencias

```
pip install -r requirements.txt
```

### Parte 3
> Comprobar que se usará una base de datos sqlite. ¿Qué fichero tienes que consultar?

`settings.py`

El bloque con la información que necesitamos es:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

> ¿Cómo se llama la base de datos que vamos a crear?

`db.sqlite3`

### Parte 4
> Crear la base de datos  
*(A partir de los modelos de datos se crean las tablas de la bd)*

```
python3 manage.py migrate
```

### Parte 5
> Crear usuario administrador

```
python3 manage.py createsuperuser
```
```
Username (leave blank to use 'vagrant'): admin
Email address:
Password:
Password (again):
Superuser created successfully.
```

- Usuario: admin
- Contraseña: 123admin456

### Parte 6
> modificar allowed hosts

`settings.py`:
```
ALLOWED_HOSTS = ['192.168.121.102']
```

Para que permita esa dirección de acceso al servidor

> Ejecuta el servidor web de desarrollo

```
python3 manage.py runserver 0.0.0.0:8000
```

Página principal:
![](https://i.imgur.com/OeNUoxK.png)


y entra en la zona de administración (/admin) Zona de administración:
![](https://i.imgur.com/filgcVV.png)


Comprobar que los datos se han añadido correctamente (usuario admin)

![](https://i.imgur.com/ZIdaCXy.png)

### Parte 7
> Crear dos preguntas, con posibles respuestas

Pregunta 1:

![](https://i.imgur.com/nlobFJe.png)

Pregunta 2:

![](https://i.imgur.com/43J3Wlb.png)

### Parte 8
> Comprobar en el navegador que la aplicación está funcionando, accede a la url /polls

Polls principal:

![](https://i.imgur.com/BKlJSAA.png)

Poll 1:

![](https://i.imgur.com/9W3HsNG.png)

Poll 2:

![](https://i.imgur.com/00l7pcH.png)





## Tarea 2: Entorno de producción

Vamos a realizar el despliegue de nuestra aplicación en un entorno de producción, para ello vamos a utilizar nuestro VPS, sigue los siguientes pasos


### Parte 1
> Clona el repositorio en el VPS.

### Parte 2
> Crea un entorno virtual e instala las dependencias de tu aplicación.

### Parte 3
> Instala el módulo que permite que python trabaje con mysql:
```
(env)$ pip install mysqlclient
```

### Parte 4
> Crea una base de datos y un usuario en mysql.

### Parte 5
> Configura la aplicación para trabajar con mysql, para ello modifica la configuración de la base de datos en el archivo settings.py:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'nombre base datos',
        'USER': 'nombre usuario',
        'PASSWORD': 'contraseña usuario',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

### Parte 6
> Como en la tarea 1, realiza la migración de la base de datos que creará la estructura de datos necesarias. Comprueba que se han creado la base de datos y las tablas.

### Parte 7
> Crea un usuario administrador.

### Parte 8
> Elige un servidor de aplicaciones python y configura nginx como proyx inverso para servir la aplicación.

### Parte 9
> Debes asegurarte que el contenido estático se está sirviendo: ¿Se muestra la imagen de fondo de la aplicación? ¿Se ve de forma adecuada la hoja de estilo de la zona de administración?.

### Parte 10
> Desactiva en la configuración el modo debug a False. Para que los errores de ejecución no den información sensible de la aplicación.

### Parte 11
> Muestra la página funcionando. En la zona de administración se debe ver de forma adecuada la hoja de estilo.

### Parte 12
> En este momento, muestra al profesor la aplicación funcionando. Entrega una documentación resumida donde expliques los pasos fundamentales para realizar esta tarea y pantallazos donde sevea que todo está funcionando.
