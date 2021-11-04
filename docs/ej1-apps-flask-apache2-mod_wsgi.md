# Ejercicio 1: Desplegando aplicaciones flask con apache2 + mod_wsgi

## ACCESO DESDE SERVER DESARROLLO

Creación del venv, activándolo, instalando requirements, instalando redis para que funcione la aplicación.

En el venv se instalan sólo paquetes python
```
1  git clone https://github.com/josedom24/guestbook.git
2  sudo apt update
3  sudo apt install https://github.com/josedom24/guestbook.git
4  git clone https://github.com/josedom24/guestbook.git
5  sudo apt install git
6  git clone https://github.com/josedom24/guestbook.git
7  ls -la
8  cd guestbook/
9  ls -la
10  cd app/
11  ls -la
12  cat requirements.txt
13  python3 -m venv
14  python3 -m venv .
15  sudo apt-get install python3-venv
16  ls -la
17  python3 -m venv .
18  ls -la
19  cd ..
20  cd .
21  cd ..
22  ls -la
23  rm -r guestbook/
24  sudo rm -r guestbook/
25  ls -la
26  python3 -m venv .
27  ls -la
28  rm -r bin/ include/ lib lib64 pyvenv.cfg share/
29  ls -al
30  mkdir venv
31  ls -la
32  cd venv/
33  python3 -m venv .
34  ls -la
35  cd ..
36  ls -la
37  git clone https://github.com/josedom24/guestbook.git
38  ls -la
39  cd guestbook/app/
40  ls -la
41  source ~/venv/bin/activate
42  pip install -r requirements.txt
43  ls -la
44  python3 app.py
45  ls -la
46  cat app.py
47  python3 app.py
48  sudo apt install redis
49  python3 app.py
50  hisory
51  history
```



## ACCESO DESDE APACHE
Módulo:
```
sudo apt install libapache2-mod-wsgi-py3
```



SÓLO HAY QUE DOCUMENTAR EL FUNCIONAMIENTO DE GUESTBOOOK
