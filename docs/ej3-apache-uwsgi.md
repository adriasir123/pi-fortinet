# Ejercicio 3: Apache + uwsgi

## Creación del venv

```
mkdir venv
sudo apt update
sudo apt install python3-venv
python3 -m venv .
source ~/venv/bin/activate
```

## Instalación de uwsgi

```
sudo apt install libpython3.9-dev build-essential
pip install uwsgi
```

## Descarga de la app guestbook

```
sudo apt install git
git clone https://github.com/josedom24/guestbook.git
pip install -r requirements.txt
sudo apt install redis
```

## Despliegue de guestbook con uwsgi

Crear `wsgi.py`:[uwsgi]
http = :8080
chdir = /home/vagrant/guestbook/app/
wsgi-file = wsgi.py
processes = 4
threads = 2
```
from app import prog as application
```

Lanzar uwsgi:
```
uwsgi --http :8080 --chdir /home/vagrant/guestbook/app/ --wsgi-file wsgi.py --process 4 --threads 2 --master
```

Creo `/home/vagrant/venv/guestbook.ini` para un lanzado más cómodo de la app:
```
[uwsgi]
http = :8080
chdir = /home/vagrant/guestbook/app/
wsgi-file = wsgi.py
processes = 4
threads = 2
```

Se lanza así:
```
uwsgi guestbook.ini
```

## Creación de una unidad systemd

`/etc/systemd/system/uwsgi-guestbook.service`
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

ExecStart=/home/vagrant/venv/bin/uwsgi /home/vagrant/venv/guestbook.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/home/vagrant/guestbook/app
Environment=PYTHONPATH='/home/vagrant/guestbook/app:/home/vagrant/venv/lib/python3.9/site-packages'

PrivateTmp=true
```

Habilitar:

```
sudo systemctl enable uwsgi-guestbook.service
sudo systemctl start uwsgi-guestbook.service
```

## Nginx como proxy para uwsgi

```
sudo apt install curl gnupg2 ca-certificates lsb-release
sudo apt install nginx
sudo nginx -t
```

VirtualHost:
`/etc/nginx/sites-available/guestbook.conf`
```
server {

    listen 80;
    server_name www.guestbook-uwsgi.com;

    location / {
        proxy_pass http://localhost:8080;
        include proxy_params;
    }

}
```

Lo habilito:
```
sudo ln -s /etc/nginx/sites-available/guestbook.conf /etc/nginx/sites-enabled/
```

Reinicio Nginx:
```
sudo systemctl restart nginx
```

Modifico mi `/etc/hosts`:
```
# ej3-apache-uwsgi resolutions
192.168.121.190 www.guestbook-uwsgi.com
```

Muestro que sigue funcionando:

![](https://i.imgur.com/xacV1bm.png)
