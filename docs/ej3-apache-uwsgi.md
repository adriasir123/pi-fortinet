# Ejercicio 3: Desplegando aplicaciones flask con Apache/Nginx + uwsgi

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

## Fichero wsgi

`/home/vagrant/guestbook/app/wsgi.py`
```
from app import prog as application
```

## Instalar uwsgi

```
sudo apt install libpython3.9-dev build-essential
pip install uwsgi
```

## Probar funcionamiento

```
uwsgi --http :8080 --chdir /home/vagrant/guestbook/app --wsgi-file wsgi.py --process 4 --threads 2 --master
```

![](https://i.imgur.com/akQFxpz.png)

Creo `/home/vagrant/guestbook/app/guestbook.ini`:
```
[uwsgi]
http = :8080
chdir = /home/vagrant/guestbook/app
wsgi-file = wsgi.py
processes = 4
threads = 2
```

Ejecuto:
```
uwsgi guestbook.ini
```

Pruebo que funciona:

![](https://i.imgur.com/acsbmco.png)

## Crear unidad systemd

`/etc/systemd/system/uwsgi-guestbook.service`:
```
[Unit]
Description=uwsgi-guestbook
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
User=www-data
Group=www-data
Restart=always

ExecStart=/home/vagrant/venv/bin/uwsgi /home/vagrant/guestbook/app/guestbook.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/home/vagrant/guestbook/app
Environment=PYTHONPATH='/home/vagrant/guestbook/app:/home/vagrant/venv/lib/python3.9/site-packages'

PrivateTmp=true
```

La activo e inicio:
```
sudo systemctl enable uwsgi-guestbook.service
sudo systemctl start uwsgi-guestbook.service
```

Pruebo que funciona:

![](https://i.imgur.com/ODtGcoJ.png)

## Apache + uWSGI

```
sudo apt install apache2
sudo a2enmod proxy_http
```

Creo `/etc/apache2/sites-available/guestbook.conf`:
```
<VirtualHost *:80>

        ServerName www.guestbook-uwsgi.com
        DocumentRoot /home/vagrant/guestbook/app

        <Directory /home/vagrant/guestbook/app>
            Require all granted
        </Directory>

        ProxyPass / http://127.0.0.1:8080/
        ProxyPassReverse / http://127.0.0.1:8080/

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
# ej3-apache-nginx-uwsgi resolutions
192.168.121.93 www.guestbook-uwsgi.com
```

Pruebo que funciona:

![](https://i.imgur.com/I0rub4s.png)

## Nginx + uWSGI

```
sudo apt install nginx
```

Creo `/etc/nginx/sites-available/guestbook.conf`:
```
server {

    listen 80;
    server_name www.guestbook-uwsgi.com;
    root /home/vagrant/guestbook/app;

    location / {
        proxy_pass http://localhost:8080;
        include proxy_params;
    }

}
```

Lo habilito:
```
sudo ln -s /etc/nginx/sites-available/guestbook.conf /etc/nginx/sites-enabled
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Pruebo que funciona:

![](https://i.imgur.com/Cac2Nbw.png)
