# Ejercicio 2: Desplegando aplicaciones flask con apache2 + gunicorn

## Creación del venv

```
mkdir venv
sudo apt update
sudo apt install python3-venv
python3 -m venv .
source ~/venv/bin/activate
```

## Instalación de gunicorn

```
pip install gunicorn
```

## Descarga de la app guestbook

```
sudo apt install git
git clone https://github.com/josedom24/guestbook.git
pip install -r requirements.txt
sudo apt install redis
```

## Despliegue de guestbook con gunicorn

Crear `wsgi.py`:
```
from app import prog as application
```

Lanzar gunicorn:
```
gunicorn -w 2 -b :8080 wsgi:application
http://192.168.121.63:8080/
```

## Creación de una unidad systemd

`/etc/systemd/system/gunicorn-guestbook.service`
```
[Unit]
Description=gunicorn-guestbook
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
User=www-data
Group=www-data
Restart=always

ExecStart=/home/vagrant/venv/bin/gunicorn -w 2 -b :8080 wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/home/vagrant/guestbook/app
Environment=PYTHONPATH='/home/vagrant/guestbook/app:/home/vagrant/venv/lib/python3.9/site-packages'

PrivateTmp=true
```

Habilitar:

```
sudo systemctl enable gunicorn-guestbook.service
sudo systemctl start gunicorn-guestbook.service
```

## Apache como proxy para gunicorn


```
sudo apt install apache2
sudo a2enmod proxy_http
```

`/etc/apache2/sites-available/guestbook.conf`:
```
<VirtualHost *:80>

	ServerName www.guestbook-gunicorn.com

        ProxyPass / http://127.0.0.1:8080/
        ProxyPassReverse / http://127.0.0.1:8080/

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
Necesitamos `ProxyPassReverse` porque Guestbook hace una redirección a localhost .


```
sudo a2ensite guestbook.conf
sudo systemctl restart apache2.service
```

```
# ej2-apache2-gunicorn resolutions
192.168.121.63 www.guestbook-gunicorn.com
```

## Nginx como proxy para gunicorn

```
sudo apt install curl gnupg2 ca-certificates lsb-release
sudo apt install nginx
sudo nginx -t
```

VirtualHost:
`/etc/nginx/conf.d/guestbook.conf`
```
server {

listen 80;
server_name     www.guestbook-gunicorn.com;

location / {
    proxy_pass http://localhost:8080;
    include proxy_params;
}

}
```

```
sudo systemctl restart nginx.service
```
