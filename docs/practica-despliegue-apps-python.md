# Práctica: Despliegue de aplicaciones python

En esta práctica se usa la aplicación [Django tutorial](https://docs.djangoproject.com/en/3.2/intro/tutorial01/).

## Tarea 1: Entorno de desarrollo

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
> Hacer fork de <https://github.com/josedom24/django_tutorial>

![](https://i.imgur.com/eqNYbrr.png)

> Clonar repo

```
git clone git@github.com:adriasir123/django_tutorial.git
```

### Parte 2
> Crear entorno virtual

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
> Comprobar que se usa SQLite. ¿Qué fichero la define?

`settings.py`, en el siguiente bloque:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

> ¿Nombre de la BD que se creará?

`db.sqlite3`

### Parte 4
> Crear tablas en SQLite

```
python3 manage.py migrate
```

### Parte 5
> Crear usuario admin

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

- **Username**: admin
- **Password**: 123admin456

### Parte 6
> Habilitar acceso a la app

`settings.py`:
```
ALLOWED_HOSTS = ['192.168.121.102']
```
*(Dirección de destino a habilitar)*

> Ejecutar servidor web desarrollo

```
python3 manage.py runserver 0.0.0.0:8000
```

Index:

![](https://i.imgur.com/OeNUoxK.png)

/admin:

![](https://i.imgur.com/filgcVV.png)

Comprobar que el usuario admin se ha añadido correctamente:

![](https://i.imgur.com/ZIdaCXy.png)

### Parte 7
> Crear dos preguntas con respuestas

Pregunta 1:

![](https://i.imgur.com/nlobFJe.png)

Pregunta 2:

![](https://i.imgur.com/43J3Wlb.png)

### Parte 8
> Comprobar que la URL `/polls` funciona

/polls:

![](https://i.imgur.com/BKlJSAA.png)

Poll 1:

![](https://i.imgur.com/9W3HsNG.png)

Poll 2:

![](https://i.imgur.com/00l7pcH.png)






## Tarea 2: Entorno de producción (VPS)

### Parte 1
> Clonar repositorio en VPS

```
git clone https://github.com/adriasir123/django_tutorial.git
```

### Parte 2
> Crear entorno virtual

```
mkdir venv
cd venv
sudo apt install python3-venv
python3 -m venv .
source bin/activate
```

> Instalar dependencias

```
pip install -r requirements.txt
```

### Parte 3
> Instalar módulo para que python se pueda comunicar con MariaDB

```
sudo apt install python3-dev default-libmysqlclient-dev build-essential
pip install mysqlclient
```

### Parte 4
> Crear BD

```
CREATE DATABASE `django_tutorial_db`;
```

> Crear usuario con permisos sobre `django_tutorial_db`

```
CREATE USER 'django_tutorial_user' IDENTIFIED BY '1234';

GRANT USAGE ON *.* TO 'django_tutorial_user'@localhost IDENTIFIED BY '1234';

GRANT ALL privileges ON `django_tutorial_db`.* TO 'django_tutorial_user'@localhost;

FLUSH PRIVILEGES;
```

### Parte 5
> Modificar la conexión a la BD en `settings.py`

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django_tutorial_db',
        'USER': 'django_tutorial_user',
        'PASSWORD': '1234',         
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

### Parte 6
> Crear tablas en MariaDB

```
python3 manage.py migrate
```

> Comprobar que se han creado

```
MariaDB [django_tutorial_db]> show tables;
+------------------------------+
| Tables_in_django_tutorial_db |
+------------------------------+
| auth_group                   |
| auth_group_permissions       |
| auth_permission              |
| auth_user                    |
| auth_user_groups             |
| auth_user_user_permissions   |
| django_admin_log             |
| django_content_type          |
| django_migrations            |
| django_session               |
| polls_choice                 |
| polls_question               |
+------------------------------+
12 rows in set (0.001 sec)
```

> Volcado de datos en desarrollo

```
python3 manage.py dumpdata > db.json
```

> Volcado de datos en VPS:

```
python3 manage.py loaddata db.json
```

### Parte 7
> Instalar y configurar Gunicorn para ejecutar Django.  
Luego, configurar Nginx como proxy inverso para servir `django_tutorial`

#### Instalar Gunicorn

```
pip install gunicorn
```

#### Crear unidad systemd

`/etc/systemd/system/gunicorn-django-tutorial.service`:
```
[Unit]
Description=gunicorn-django-tutorial
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
User=www-data
Group=www-data
Restart=always

ExecStart=/home/blackmamba/venv/bin/gunicorn -w 2 -b :8080 django_tutorial.wsgi
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/home/blackmamba/django_tutorial
Environment=PYTHONPATH='/home/blackmamba/django_tutorial:/home/blackmamba/venv/lib/python3.9/site-packages'

PrivateTmp=true
```

La activo e inicio:
```
sudo systemctl enable gunicorn-django-tutorial.service
sudo systemctl start gunicorn-django-tutorial.service
```

#### Nginx + Gunicorn

Creo `/etc/nginx/sites-available/django.conf`:
```
server {

    listen 80;
    server_name django.adrianjaramillo.tk;
    root /home/blackmamba/django_tutorial;

    location / {
        proxy_pass http://localhost:8080;
        include proxy_params;
    }

}
```

Lo habilito:
```
sudo ln -s /etc/nginx/sites-available/django.conf /etc/nginx/sites-enabled/
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

#### Añadir nuevo registro DNS

![](https://i.imgur.com/Dgw0mq8.png)

#### Modificar settings.py

```
ALLOWED_HOSTS = ['django.adrianjaramillo.tk']
```

Reinicio:
```
sudo systemctl restart gunicorn-django-tutorial.service
```

Pruebo que funciona:

![](https://i.imgur.com/ag517YK.png)

### Parte 8
> ¿Funciona el css de polls? Arréglalo.

`/etc/nginx/sites-available/django.conf`:
```
location /static/polls {
    alias /home/blackmamba/django_tutorial/polls/static/polls;
}
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Pruebo que funciona:

![](https://i.imgur.com/2SGXgU1.png)

> ¿Funciona el css de admin? Arréglalo.

`/etc/nginx/sites-available/django.conf`:
```
location /static/admin {
    alias /home/blackmamba/venv/lib/python3.9/site-packages/django/contrib/admin/static/admin/;
}
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Pruebo que funciona:

![](https://i.imgur.com/plc3TzP.png)

### Parte 9
> Desactivar debug

`settings.py`:
```
DEBUG = False
```

Reinicio:
```
sudo systemctl restart gunicorn-django-tutorial.service
```

Pruebo que funciona:

![](https://i.imgur.com/ESRxqoB.png)

Si DEBUG fuese True, al escribir esa URL que no existe, me habría aparecido mucha más información:

![](https://i.imgur.com/Gt9qRe9.png)

*(DEBUG = True)*

### Parte 10
> Mostrar la página funcionando

![](https://i.postimg.cc/5ydStsgg/djangotutorial-vps-funcionando.gif)






## Tarea 3: Modificación de nuestra aplicación

En esta sección primero se cambia en desarrollo, y luego se pasa a producción.

### Parte 0
> Añadir `settings.py` a `.gitignore`

```
django_tutorial/settings.py
```

**¡Esta acción es importante!**  
El fichero cambia MUCHO con respecto a desarrollo/producción *(DEBUG, ALLOWED_HOSTS, DATABASES)*.

### Parte 1
> Modificar `django_tutorial/polls/templates/polls/index.html` con tu nombre

```
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
    <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}

<h2>Adrián Jaramillo Rodríguez</h2>
```

Prueba en desarrollo:

![](https://i.postimg.cc/Xq4nw8Zj/polls-nombre.png)

Subir cambios desde desarrollo:
```
git commit -am "mi nombre polls index.html"
git push
```

Bajar cambios en producción:
```
git pull
```

Reinicio gunicorn para que actualice la configuración:
```
sudo systemctl restart gunicorn-django-tutorial.service
```

Prueba en producción:

![](https://i.postimg.cc/DzD9R2yD/minombre-polls-produccion.png)

### Parte 2
> Modificar la imagen de fondo en la página principal

En `/home/vagrant/django_tutorial/polls/templates/index.html`...

Añadir en el head:
```
<style>
  body {
    background-image: url('https://w.wallhaven.cc/full/lm/wallhaven-lm12ey.jpg');
  }
</style>
```

Comentar en el body:
```
<!--
  <div id="stripes">
    <span></span>
    <span></span>
    <span></span>
    <span></span>
    <span></span>
  </div>
-->
```

Prueba en desarrollo:

![](https://i.imgur.com/w8Jy6Mq.png)

Subir cambios desde desarrollo:
```
git commit -am "fondo cambiado en index.html"
git push
```

Bajar cambios en producción:
```
git pull
```

Reinicio gunicorn para que actualice la configuración:
```
sudo systemctl restart gunicorn-django-tutorial.service
```

Prueba en producción:

![](https://i.imgur.com/bzbYZpn.png)

### Parte 3
> Crear una nueva tabla en la base de datos

Añadir a `django_tutorial/polls/models.py`:
```
class Categoria(models.Model):
    Abr = models.CharField(max_length=4)
    Nombre = models.CharField(max_length=50)

    def __str__(self):
  	    return self.Abr+" - "+self.Nombre
```

Aplicar cambios:
```
python manage.py makemigrations
python manage.py migrate
```

Modificar la siguiente línea en `django_tutorial/polls/admin.py`...
```
from .models import Choice, Question, Categoria
```

...y añadir esta otra:
```
admin.site.register(Categoria)
```

Prueba en desarrollo:

![](https://i.imgur.com/qozi4Oh.png)
*(la tabla categorías se ha creado, y he rellenado datos)*

Vuelco los nuevos datos en desarrollo:
```
python3 manage.py dumpdata > db2.json
```

Subir cambios desde desarrollo:
```
git add .
git commit -am "nueva tabla"
git push
```

Bajar cambios en producción:
```
git pull
```

Creo la tabla:
```
python manage.py migrate
```

Cargo los datos:
```
python3 manage.py loaddata db2.json
```

Reinicio gunicorn para que actualice la configuración:
```
sudo systemctl restart gunicorn-django-tutorial.service
```

Prueba en producción:

![](https://i.imgur.com/nnNjlC7.png)
