# Práctica: Instalación de un CMS python

## Apartado 1
> Instalar Django Fiber en desarrollo, utilizando un entorno virtual

Escenario:
```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :practicacmspython do |practicacmspython|
    practicacmspython.vm.box = "rockylinux/8"
    practicacmspython.vm.hostname = "practicacmspython"
  end

end
```

Creo entorno virtual:
```
mkdir venv
cd venv
sudo dnf install virtualenv
python3 -m venv .
source bin/activate
```

Descargo el proyecto:
```
sudo dnf install git
git clone https://github.com/django-fiber/django-fiber.git
```

Instalo paquetes necesarios:
```
sudo dnf groupinstall 'Development Tools'
pip install --upgrade pip
pip install -r requirements.txt
pip install django-fiber
```

Modifico `settings.py`:
```
ALLOWED_HOSTS = ['192.168.121.54']
```

Creo las tablas:
```
python3 manage.py migrate
```

Creo el usuario admin:
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

Ejecuto el server web de desarrollo:
```
python3 manage.py runserver 0.0.0.0:8000
```

Pruebo que funciona:

![](https://i.imgur.com/thnsB2Q.png)

## Apartado 2
> Personalizar la página

Para ello es necesario acceder a `/admin`.

Principal:

![](https://i.imgur.com/hkfemvp.png)

Artículo 1:

![](https://i.imgur.com/1HJrgIj.png)

## Apartado 3
> Volcar la base de datos  

```
python manage.py dumpdata --exclude auth.permission --exclude contenttypes > db.json
```

> Guardar la aplicación en un repositorio

Lo creo: <https://github.com/adriasir123/django-fiber>

Dejo `.gitignore` así:
```
*.pyc
/build/
/dist/
/testproject/reports/
#/testproject/static/
django_fiber.egg-info
.tox

testproject/fiber_test/__pycache__
testproject/testproject/__pycache__
```

Subo al repo:
```
git remote remove origin
git branch -M main
git add .
git config --global user.email "adristudy@gmail.com"
git config --global user.name "Adrián Jaramillo"
git commit -m "Initial commit"
git push --set-upstream origin master
```

## Apartado 4
> Desplegar la app en hera (servidor web apache y servidor de base de datos mariadb en KVM), utilizando un entorno virtual. Utiliza uwsgi. El contenido estático debe servirlo el servidor web. La aplicación será accesible en la url python.adrianj.gonzalonazareno.org.

Clonar repositorio:
```
sudo dnf install git
git clone https://github.com/adriasir123/django-fiber.git
```

Creo entorno virtual:
```
mkdir venv
cd venv
sudo dnf install virtualenv
python3 -m venv .
source bin/activate
```

Instalo paquetes necesarios:
```
sudo dnf groupinstall 'Development Tools'
pip install --upgrade pip
pip install -r requirements.txt
pip install django-fiber
```

Instalar módulo para que python se pueda comunicar con MariaDB:
```
sudo dnf install python3-devel mysql-devel
pip install mysqlclient
```

Crear BD:
```
CREATE DATABASE `django_fiber_db`;
```

Crear usuario con permisos sobre `django_fiber_db`:
```
CREATE USER 'django_fiber_user' IDENTIFIED BY '1234';

GRANT USAGE ON *.* TO 'django_fiber_user'@'%' IDENTIFIED BY '1234';

GRANT ALL privileges ON `django_fiber_db`.* TO 'django_fiber_user'@'%';

FLUSH PRIVILEGES;
```

Modificar la conexión a la BD en `settings.py`:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django_fiber_db',
        'USER': 'django_fiber_user',
        'PASSWORD': '1234',         
        'HOST': 'bd.adrianj.gonzalonazareno.org',
        'PORT': '',
    }
}
```

Crear tablas en MariaDB:
```
python3 manage.py migrate
```

Recuperar de datos:
```
python3 manage.py loaddata db.json
```

Instalar uwsgi
```
pip install uwsgi
```

Creo `/var/www/django-fiber/testproject/django-fiber.ini`:
```
[uwsgi]
http = :8080
chdir = /var/www/django-fiber/testproject
wsgi-file = /var/www/django-fiber/testproject/testproject/wsgi.py
processes = 4
threads = 2
```

Crear unidad systemd, `/etc/systemd/system/uwsgi-django-fiber.service`:
```
[Unit]
Description=uwsgi-django-fiber
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
User=root
Group=root
Restart=always

ExecStart=/home/vagrant/venv/bin/uwsgi /var/www/django-fiber/testproject/django-fiber.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/var/www/django-fiber/testproject
Environment=PYTHONPATH='/var/www/django-fiber/testproject:/home/vagrant/venv/lib/python3.6/site-packages'

PrivateTmp=true
```

Cambio permisos:
```
sudo chown -R apache:apache /home/vagrant/django-fiber/
sudo chown -R apache:apache /home/vagrant/venv
sudo chmod 777 django-fiber.ini
```

Deshabilitar selinux:
```
sudo setenforce 0
```


La activo e inicio:
```
sudo systemctl enable uwsgi-django-fiber.service
sudo systemctl start uwsgi-django-fiber.service
```

### Apache + uWSGI

```
python manage.py collectstatic --link
```

Creo `/etc/httpd/conf.d/python.conf`:
```
<VirtualHost *:80>

        ServerName python.adrianj.gonzalonazareno.org
        DocumentRoot /var/www/django-fiber/testproject

        <Directory /var/www/django-fiber/testproject>
            Require all granted
            Options +FollowSymLinks
        </Directory>

        ProxyPass /static/ !
        ProxyPass / http://127.0.0.1:8080/
        ProxyPassReverse / http://127.0.0.1:8080/

</VirtualHost>
```

Mover la app:
```
sudo mv django-fiber/ /var/www/
```

Permisos:
```
sudo chown -R root:root /var/www/django-fiber/
```


Reinicio Apache:
```
sudo systemctl restart httpd
```

### Nuevos registro DNS

Añado a `db.externa.adrianj.gonzalonazareno.org`:
```
python         IN    CNAME   zeus
```

Añado a `db.dmz.adrianj.gonzalonazareno.org`:
```
python         IN    CNAME   hera
```

Añado a `db.interna.adrianj.gonzalonazareno.org`:
```
python         IN    CNAME   hera
```

Reiniciar bind9:
```
sudo systemctl restart bind9
```


### Modifico puntos de entrada

```
ALLOWED_HOSTS = ['python.adrianj.gonzalonazareno.org','127.0.0.1']
```

Reinicio uwsgi:
```
sudo systemctl restart uwsgi-django-fiber.service
```
