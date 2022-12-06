# Ejercicio 1: Desplegando aplicaciones flask con Apache + mod_wsgi

En esta tarea se usa la aplicación [Guestbook](https://github.com/josedom24/guestbook).

## Crear venv

```
mkdir venv
cd venv
sudo apt update
sudo apt install python3-venv
python3 -m venv .
source bin/activate
```

## Descargar Guestbook

```
sudo apt install git
git clone https://github.com/josedom24/guestbook.git
pip install -r requirements.txt
sudo apt install redis
```

## Probar funcionamiento

Por ahora, usando el servidor de desarrollo:
```
python3 app.py
```

![](https://i.imgur.com/VRc81fd.png)

## Fichero wsgi

`/home/vagrant/guestbook/app/wsgi.py`
```
from app import prog as application
```

## Hacer que Apache sirva Guestbook

```
sudo apt install apache2
sudo apt install libapache2-mod-wsgi-py3
```

Creo `/etc/apache2/sites-available/guestbook.conf`:
```
<VirtualHost *:80>

        ServerName www.guestbook.com
        DocumentRoot /home/vagrant/guestbook/app

        <Directory /home/vagrant/guestbook/app>
            Require all granted
        </Directory>

        WSGIDaemonProcess guestbook python-path=/home/vagrant/guestbook/app:/home/vagrant/venv/lib/python3.9/site-packages
        WSGIProcessGroup guestbook
        WSGIScriptAlias / /home/vagrant/guestbook/app/wsgi.py process-group=guestbook

</VirtualHost>
```

Lo habilito:
```
sudo a2ensite guestbook.conf
```

Reinicio Apache:
```
sudo systemctl restart apache2
```

Modifico mi `/etc/hosts`:
```
# ej1-apps-flask-apache2-mod_wsgi resolutions
192.168.121.136 www.guestbook.com
```

## Probar funcionamiento Apache

![](https://i.imgur.com/w6HPB4v.png)
