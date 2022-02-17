# Práctica: Servidor de correos en VPS

## Pasos previos

Instalo postfix:

```console
sudo apt update
sudo apt install postfix
```

Selecciono "Internet Site":

![postfix install modes](https://i.imgur.com/AQArWhQ.png)

Escribo mi nombre de dominio. Se tomará como dominio origen de los correos que se envíen desde el vps *(`myorigin = /etc/mailname`)*:

![dominio origen](https://i.imgur.com/RSQA07R.png)

Instalo `mailutils`:

```console
sudo apt install mailutils
```

## Gestión de correos desde el servidor

### Tarea 1

#### 1.1

> Crear registro SPF en el DNS

![registro spf](https://i.imgur.com/Fmp7QUO.png)

#### 1.2

> Enviar un correo desde la vps a gmail

```console
blackmamba@kampe:~$ mail -r blackmamba@adrianjaramillo.tk adristudy@gmail.com
Cc:
Subject: Correo con SPF
Mensaje

```

#### 1.3

> Mostrar `/var/log/mail.log` para verificar que el correo se ha enviado correctamente

Línea relevante:

```console
Feb 14 16:20:33 kampe postfix/smtp[516217]: 242F742268: to=<adristudy@gmail.com>, relay=gmail-smtp-in.l.google.com[64.233.167.26]:25, delay=1.3, delays=0.03/0.01/0.29/0.95, dsn=2.0.0, status=sent (250 2.0.0 OK  1644855633 l11si9233120wms.107 - gsmtp)
```

#### 1.4

> Mostrar el correo recibido en gmail

![correo recibido](https://i.imgur.com/paotFxI.png)

Para estar seguros de que este correo ha llegado con SPF válido, tenemos que mostrar el correo original:

![checkeo pass válido](https://i.imgur.com/cUi9Kb7.png)

Ha pasado el checkeo SPF, pero si queremos ver información aún más específica, podemos hacer scroll al contenido en bruto del correo recibido:

![spf original](https://i.imgur.com/KSzFK0e.png)

Al ver eso, queda totalmente claro que el checkeo SPF ha sido exitoso.

### Tarea 2

#### 2.1

> Crear registro MX y CNAME correspondiente en el DNS

![registros dns mx y mail](https://i.imgur.com/1pcAAMz.png)

#### 2.2

> Habilitar en el firewall del vps el tráfico entrante al puerto 25

![firewall 25 vps](https://i.imgur.com/r6VLM8T.png)

#### 2.3

> Enviar un correo desde gmail a la vps

![correo para vps](https://i.imgur.com/3duHQSd.png)

#### 2.4

> Comprobar que la vps ha recibido correctamente el correo mirando `/var/log/mail.log`

![mail log vps](https://i.imgur.com/zfKjEvb.png)

#### 2.5

> Mostrar el correo recibido

```console
blackmamba@kampe:~$ mail
"/var/mail/blackmamba": 1 message 1 new
>N   1 adristudy          Mon Feb 14 18:17  52/2630  Correo para la VPS
? 1
Return-Path: <adristudy@gmail.com>
X-Original-To: blackmamba@adrianjaramillo.tk
Delivered-To: blackmamba@adrianjaramillo.tk
Received: from mail-pl1-f176.google.com (mail-pl1-f176.google.com [209.85.214.176])
	by kampe.adrianjaramillo.tk (Postfix) with ESMTPS id 4F393422CB
	for <blackmamba@adrianjaramillo.tk>; Mon, 14 Feb 2022 18:17:40 +0000 (UTC)
Received: by mail-pl1-f176.google.com with SMTP id l8so5186364pls.7
        for <blackmamba@adrianjaramillo.tk>; Mon, 14 Feb 2022 10:17:40 -0800 (PST)
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=gmail.com; s=20210112;
        h=mime-version:from:date:message-id:subject:to;
        bh=MudV18FBtYjZJPFyKfQ5hC6U6ERmLoC4WY7BpsZqJUs=;
        b=hpLYd8Radagaj5ilgByYq41SIVkdDFuM28I2tHi2qxE6yANOJnGC22x6FepMkW/ur3
         2iqHhmlw72u8he4u1VNxYsP3xCORcgx77iI3nxCkAYG7l9Tx6L7cScNNBMmsgfzP0Xp2
         ukKClt9Nc38F0xHSwF1qKrpie4OsxxB+blYUnUzx5SvUl18WtuHjfxLHM2cPpueYrYvc
         Ls0dR/Hyt9pWXEthi9ueHnWTprlauAdxKANUJMnbYA1C5z6YH0F+i/6NnHa96ZQ2E2Dv
         JrRzfqVYRwNwDzQAHa5ViqbgyepDPLmu9gnbytlfJCoQSrAs9diH/8gLU7b3ymnlkxQS
         HkNg==
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=1e100.net; s=20210112;
        h=x-gm-message-state:mime-version:from:date:message-id:subject:to;
        bh=MudV18FBtYjZJPFyKfQ5hC6U6ERmLoC4WY7BpsZqJUs=;
        b=VUVBAkMaD/qnhm92cnmVBYhTFj0HuNxKqtgGNEVlEge780WB2QaXPMtaSMGSz77aHc
         nC3VQvj18EtItjtccAywt/FLLqP4ucH2gcellCvn0UreOCFoP8v7RaZTMYG4RBYrCOMD
         WKGXOXO2ay9fsCIQSNp429tQnRumifl3XaVMfy3jSZAbt1HUghtRP2VgE6Y/SLz5Lmkq
         NKEFW1u4IHqk8aXvM1cBzD6szJvPs5LSJjkmNuq/k/P+kH1QWQhrvs85Lyxzfgo3RSj+
         qIG6+H1TK4JI3bBMbhwEiiapEJOVYj2DsGmeSoDTvNChGryidxJMQqpDTdz3mZ+ZE37K
         bWxA==
X-Gm-Message-State: AOAM530YXJN7SPi6YOZ/UjMpMOMY71KTCE4ZLpC/AcW60wbzOkjuyN9M
	fWHiGc3H9WbfeAMOAzOBkY8vtybUr+Pw96kZMXAREDN/
X-Google-Smtp-Source: ABdhPJz6Vyx6fBNDRDGMOl88iRuxk0moB4G3eJTCNLmoXHc1ZDYlXFgmJdhK8aHYgutvtMLZmiO1TJ06tqf6XTu1iHA=
X-Received: by 2002:a17:90a:2e0a:: with SMTP id q10mr1050878pjd.130.1644862658470;
 Mon, 14 Feb 2022 10:17:38 -0800 (PST)
MIME-Version: 1.0
From: adristudy <adristudy@gmail.com>
Date: Mon, 14 Feb 2022 19:17:27 +0100
Message-ID: <CAG5uPRccDy2iqjcOTT29ekY6tNCcnsgy0VsA5pXjzdm7KCcLgw@mail.gmail.com>
Subject: Correo para la VPS
To: blackmamba@adrianjaramillo.tk
Content-Type: multipart/alternative; boundary="00000000000076676e05d7fe6ef6"

--00000000000076676e05d7fe6ef6
Content-Type: text/plain; charset="UTF-8"

Prueba

--00000000000076676e05d7fe6ef6
Content-Type: text/html; charset="UTF-8"

<div dir="ltr">Prueba</div>

--00000000000076676e05d7fe6ef6--
```

## Tarea 3: Uso de alias y redirecciones

### 3.1

> Crear tarea cron

Abro la configuración:

```console
crontab -e
```

Añado lo siguiente:

```console
MAILTO = root

* * * * * echo Prueba de mensajes crontab
```

Esta tarea ejecuta un echo cada minuto y envía un correo a root con el resultado.  
No vemos ese echo por terminal ni en la sesión de nuestro usuario ni en la de root, pero nos sirve para hacer pruebas.

### 3.2

> Comprobar que llegan correos en root

```console
root@kampe:~# mail
"/var/mail/root": 64 messages 64 unread
>U   1 Cron Daemon        Mon Feb 14 21:20  23/788   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
 U   2 Cron Daemon        Mon Feb 14 21:21  22/762   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
 U   3 Cron Daemon        Mon Feb 14 21:22  22/762   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
 U   4 Cron Daemon        Mon Feb 14 21:23  22/762   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
```

Son todos iguales, así que muestro uno de ellos:

![correo crontab root](https://i.imgur.com/dWPZ8k3.png)

La parte que nos interesa es la marcada, el "From" y el "To".  
Los correos los envía el daemon de Cron usando root, hacia el usuario root.  
En el cuerpo del correo está el resultado del echo.

Según como está ahora configurado cron, no nos llegarían esos correos a nuestro usuario.

### 3.3

> Crear un alias para que lleguen los correos a `blackmamba`

Modifico `/etc/aliases`:

```console
# See man 5 aliases for format
postmaster: root
root: blackmamba
```

Para que este cambio tome efecto ejecuto:

```console
sudo newaliases
```

### 3.4

> Comprobar que ya llegan los correos a `blackmamba`

```console
blackmamba@kampe:~$ mail
"/var/mail/blackmamba": 1 message 1 new
>N   1 Cron Daemon        Mon Feb 14 22:59  20/743   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
?
```

Lo abro:

```console
? 1
Return-Path: <blackmamba@adrianjaramillo.tk>
X-Original-To: root
Delivered-To: root@adrianjaramillo.tk
Received: by kampe.adrianjaramillo.tk (Postfix, from userid 1000)
	id BC48442AC0; Mon, 14 Feb 2022 22:59:01 +0000 (UTC)
From: root@adrianjaramillo.tk (Cron Daemon)
To: root@adrianjaramillo.tk
Subject: Cron <blackmamba@kampe> echo Prueba de mensajes crontab
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <MAILTO=root>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/home/blackmamba>
X-Cron-Env: <PATH=/usr/bin:/bin>
X-Cron-Env: <LOGNAME=blackmamba>
Message-Id: <20220214225901.BC48442AC0@kampe.adrianjaramillo.tk>
Date: Mon, 14 Feb 2022 22:59:01 +0000 (UTC)

Prueba de mensajes crontab
?
```

He mostrado el contenido para que se vea el campo "To:", que apunta a root, pero el alias funciona así que me llega a mi usuario principal `blackmamba`.

### 3.5

> Crear una redirección para enviar esos correos a gmail

Creo `/home/blackmamba/.forward` con el siguiente contenido:

```console
adristudy@gmail.com
```

Veo en gmail que me están llegando:

![correos cron a gmail](https://i.imgur.com/CV4QGD4.png)

## Gestión de correos desde un cliente

### Tarea 8

#### 8.1

> Borrar el fichero `.forward` anteriormente creado

```console
rm .forward
```

Es **obligatorio** que borremos este fichero para que lo que hagamos durante esta tarea funcione, y los correos lleguen a `Maildir` correctamente.

¿Por qué? El fichero `.forward` tiene el siguiente funcionamiento:

![funcionamiento forward](https://i.imgur.com/pxhEqFg.png)

Es decir, **si tenemos un fichero `.forward` no podremos recibir ningún correo en usuarios locales**.

#### 8.2

> Configurar postfix con buzón Maildir

Añado lo siguiente a `/etc/postfix/main.cf`:

```console
home_mailbox = Maildir/
```

Reinicio postfix:

```console
sudo systemctl restart postfix
```

#### 8.3

> Enviar un correo a `blackmamba`

Hago la prueba:

```console
blackmamba@kampe:~$ mail -r blackmamba@localhost blackmamba@localhost
Cc:
Subject: Prueba Maildir
Mensaje

```

#### 8.4

> Comprobar que el correo se ha guardado en el buzón Maildir

Primero compruebo `/var/log/mail.log` para ver que haya llegado correctamente:

![maildir log](https://i.imgur.com/lrEepw0.png)

"delivered to maildir" significa que ha funcionado correctamente.

Lo siguiente a comprobar es la estructura de `Maildir` que se nos ha generado en el home:

```console
blackmamba@kampe:~$ tree Maildir/
Maildir/
|-- cur
|-- new
|   `-- 1644944881.Vfe01I605c1M715492.kampe
`-- tmp

3 directories, 1 file
```

**¡Atención! NO TENEMOS que crear esta estructura de directorios manualmente para que funcione, lo hace postfix.**

#### 8.5

> Visualizar el correo recibido en `Maildir`

Podríamos perfectamente abrir manualmente los ficheros con `cat`, pero esto es rudimentario, usaremos la herramienta `mail`.

Según se dice en la guía de José Domingo, desde que usamos `Maildir` como buzón ya no podremos visualizar los correos con `mail`. Esto es cierto si ejecutamos el comando sin argumentos:

```console
blackmamba@kampe:~$ mail
No mail for blackmamba
```

**PERO** podemos hacer lo siguiente:

```console
blackmamba@kampe:~$ mail -f Maildir
"/home/blackmamba/Maildir": 1 message 1 new
>N   1 blackmamba@localho Tue Feb 15 17:08  13/459   Prueba Maildir
? 1
Return-Path: <blackmamba@localhost>
X-Original-To: blackmamba@localhost
Delivered-To: blackmamba@localhost
Received: by kampe.adrianjaramillo.tk (Postfix, from userid 1000)
	id A9FDB42CF6; Tue, 15 Feb 2022 17:08:01 +0000 (UTC)
To: <blackmamba@localhost>
Subject: Prueba Maildir
X-Mailer: mail (GNU Mailutils 3.10)
Message-Id: <20220215170801.A9FDB42CF6@kampe.adrianjaramillo.tk>
Date: Tue, 15 Feb 2022 17:08:01 +0000 (UTC)
From: blackmamba@localhost

Mensaje
?
```

Indicando la ruta de `Maildir` con `mail -f` nos posibilita ver los correos perfectamente.

Muestro el man de `mail` para un mejor entendimiento:

![mail -f man](https://i.imgur.com/95YeS20.png)

### Tarea 9

#### 9.1

> Instalar Dovecot para ofrecer el protocolo IMAPS

```console
sudo apt install dovecot-imapd
```

Lo primero que podemos ver es que estará escuchando en el puerto que necesitamos:

![dovecot ports](https://i.imgur.com/XOEPZJ5.png)

Tenemos que permitir ese tráfico en el firewall de la vps:

![993 firewall vps](https://i.imgur.com/Y9siAwo.png)

Hacemos una primera prueba de comunicación con el puerto desde nuestro host:

```console
atlas@olympus:~$ telnet 87.106.228.149 993
Trying 87.106.228.149...
Connected to 87.106.228.149.
Escape character is '^]'.

```

#### 9.2

> Configurar Dovecot para que use el buzón Maildir

Modifico `/etc/dovecot/conf.d/10-mail.conf`:

```console
mail_location = maildir:~/Maildir
```

Reinicio dovecot:

```console
sudo systemctl restart dovecot
```

#### 9.3

> Configurar el cifrado de la comunicación por IMAPS en Dovecot con certificado. Usar el wildcard `*.adrianjaramillo.tk` que ya teníamos.

Modifico `/etc/dovecot/conf.d/10-ssl.conf`:

```console
##
## SSL settings
##

# SSL/TLS support: yes, no, required. <doc/wiki/SSL.txt>
ssl = yes

# PEM encoded X.509 SSL/TLS certificate and private key. They're opened before
# dropping root privileges, so keep the key file unreadable by anyone but
# root. Included doc/mkcert.sh can be used to easily generate self-signed
# certificate, just make sure to update the domains in dovecot-openssl.cnf
ssl_cert = </etc/letsencrypt/live/adrianjaramillo.tk/fullchain.pem
ssl_key = </etc/letsencrypt/live/adrianjaramillo.tk/privkey.pem
```

Sólo muestro el principio del fichero, porque es lo que he modificado.

Reinicio dovecot:

```console
sudo systemctl restart dovecot
```

#### 9.4

> Configurar evolution para recibir los correos del vps

Abro la configuración para una nueva cuenta de mail:

![evolution recibir 1](https://i.postimg.cc/sD7S1bkL/evolution-inicio-config-cuenta-mail.gif)

Indico mi cuenta de correo y un nombre *(podrá ser cualquiera, evolution sólo lo usa para identificar la conexión)*:

![evolution recibir 2](https://i.imgur.com/0SLprAF.png)

Configuro la recepción de correos:

![evolution recibir 3](https://i.imgur.com/TElH0SV.png)

La autenticación se va a realizar con usuarios del sistema, por eso escribo `blackmamba`. Para que funcione la autenticación en Dovecot no he tenido que configurar nada, ya funciona por defecto.

Mantengo las opciones de recepción por defecto:

![evolution recibir 4](https://i.postimg.cc/sXNX7VnW/receiving-options-evolution.png)

Relleno las opciones de envío para que me deje terminar la configuración, pero **ESTE NO ES el objetivo de este apartado, así que la configuración estará incompleta y por supuesto, no funcional**:

![evolution recibir 5](https://i.postimg.cc/wMzwgppC/sending-evolution-nope.png)

El envío se configurará en la tarea 11.

Muestro el resumen final de la configuración, y de nuevo ignoramos la parte referente al envío:

![evolution recibir 6](https://i.postimg.cc/6QkwXYpk/account-summary-evolution-receiving.png)

Por último se nos pregunta la contraseña del usuario que hemos indicado anteriormente:

![evolution recibir 7](https://i.postimg.cc/FzjnLxmJ/authentication-evolution-receiving.png)

Compruebo que puedo leer el correo de `Maildir` que envié en la tarea anterior:

![evolution recibir 8](https://i.postimg.cc/dtBDRkqb/prueba-mostrado-maildir.gif)

Evolution se sincroniza con Dovecot de forma live, así que podríamos enviar un correo y Evolution lo recibiría automáticamente.  
Muestro la prueba:

![evolution recibir 9](https://i.postimg.cc/GhjcyWcz/evolution-dovecot-live.gif)

### Tarea 11

#### 11.1

> Configurar postfix para ofrecer el protocolo SMTPS

Descomento el siguiente bloque en `/etc/postfix/master.cf`:

```console
smtps     inet  n       -       y       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_reject_unlisted_recipient=no
  -o smtpd_client_restrictions=$mua_client_restrictions
  -o smtpd_helo_restrictions=$mua_helo_restrictions
  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

Reinicio postfix:

```console
sudo systemctl restart postfix
```

Compruebo que ahora funciona el puerto SMTPS 465:

![puerto 465 abierto](https://i.imgur.com/ZrF26R9.png)

Tenemos que permitir ese tráfico en el firewall de la vps:

![465 firewall vps](https://i.imgur.com/klheTIh.png)

Hago una primera prueba de comunicación con el puerto desde nuestro host:

```console
atlas@olympus:~$ telnet 87.106.228.149 465
Trying 87.106.228.149...
Connected to 87.106.228.149.
Escape character is '^]'.

```

#### 11.2

> Usar certificados para cifrar el envío de correos

Modifico lo siguiente en `/etc/postfix/main.cf`:

```console
smtpd_tls_cert_file=/etc/letsencrypt/live/adrianjaramillo.tk/fullchain.pem
smtpd_tls_key_file=/etc/letsencrypt/live/adrianjaramillo.tk/privkey.pem
```

Reinicio postfix:

```console
sudo systemctl restart postfix
```

#### 11.3

> Configurar autenticación SASL tanto en Postfix como en Dovecot

Añado lo siguiente a `/etc/postfix/main.cf`:

```console
# Authentication

smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
```

Reinicio postfix:

```console
sudo systemctl restart postfix
```

Descomento el siguiente bloque en `/etc/dovecot/conf.d/10-master.conf`:

```console
  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
  }
```

Reinicio dovecot:

```console
sudo systemctl restart dovecot
```

#### 11.4

> Configurar evolution para enviar correos y mostrar una prueba de envío

Borro la conexión anterior para poder crear una nueva:

![evolution enviar 1](https://i.postimg.cc/4NkrLBPj/borrar-conexion-evolution.gif)

Creo una nueva conexión, y muestro directamente la configuración de envío:

![evolution enviar 2](https://i.imgur.com/ZNVSYnR.png)

En el resumen final compruebo que la información de envío sea correcta:

![evolution enviar 3](https://i.imgur.com/NEFWz6b.png)

Envío un correo:

![evolution enviar 4](https://i.imgur.com/KjKcfQP.png)

Antes de que se realice el envío, se pide autenticación:

![evolution enviar 5](https://i.imgur.com/ZIHcSug.png)

Compruebo que me ha llegado el correo a gmail:  

![evolution enviar 6](https://i.imgur.com/z71mbM4.png)

Si queremos estar incluso más seguros de que este correo efectivamente se ha enviado desde Evolution y algún detalle más, podemos abrir el original y mostrar sus cabeceras:

![evolution enviar 7](https://i.imgur.com/R70kffQ.png)

## Tarea 13: comprobación final

> Enviar un correo a [esta página](https://www.mail-tester.com/) para obtener una puntuación. Mostrar el resultado.

Este es el correo que enviaremos:

![correo prueba calidad](https://i.imgur.com/YTpGKT2.png)

[Haz click aquí](https://www.mail-tester.com/test-0oeev9pyl) para ver mis resultados.
