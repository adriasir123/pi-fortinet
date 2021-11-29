# Ejercicio 2: Desplegando aplicaciones flask con Apache + gunicorn

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

## Instalar Gunicorn

```
pip install gunicorn
```

## Probar funcionamiento

```
gunicorn -w 2 -b :8080 wsgi
```

![](https://i.imgur.com/EfohYP9.png)

## Crear unidad systemd

`/etc/systemd/system/gunicorn-guestbook.service`:
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

ExecStart=/home/vagrant/venv/bin/gunicorn -w 2 -b :8080 wsgi
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/home/vagrant/guestbook/app
Environment=PYTHONPATH='/home/vagrant/guestbook/app:/home/vagrant/venv/lib/python3.9/site-packages'

PrivateTmp=true
```

La activo e inicio:
```
sudo systemctl enable gunicorn-guestbook.service
sudo systemctl start gunicorn-guestbook.service
```

Pruebo que funciona:

![](https://i.imgur.com/vmqHUoY.png)

## Apache + Gunicorn

```
sudo apt install apache2
sudo a2enmod proxy_http
```

Creo `/etc/apache2/sites-available/guestbook.conf`:
```
<VirtualHost *:80>

        ServerName www.guestbook-gunicorn.com
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
# ej2-guestbook-gunicorn resolutions
192.168.121.2 www.guestbook-gunicorn.com
```

Pruebo que funciona:

![](https://i.imgur.com/Bfz4n77.png)

## Nginx + Gunicorn

```
sudo apt install nginx
```

Creo `/etc/nginx/sites-available/guestbook.conf`:
```
server {

    listen 80;
    server_name www.guestbook-gunicorn.com;
    root /home/vagrant/guestbook/app;

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

Pruebo que funciona:

![](https://i.imgur.com/VFZn6WR.png)
