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

> Crear registro SPF en el DNS

![registro spf](https://i.imgur.com/Fmp7QUO.png)

> Enviar un correo desde la vps a gmail

```console
blackmamba@kampe:~$ mail -r blackmamba@adrianjaramillo.tk adristudy@gmail.com
Cc:
Subject: Correo con SPF
Mensaje

```

> Mostrar `/var/log/mail.log` para verificar que el correo se ha enviado correctamente

Muestro la línea relevante:

```console
Feb 14 16:20:33 kampe postfix/smtp[516217]: 242F742268: to=<adristudy@gmail.com>, relay=gmail-smtp-in.l.google.com[64.233.167.26]:25, delay=1.3, delays=0.03/0.01/0.29/0.95, dsn=2.0.0, status=sent (250 2.0.0 OK  1644855633 l11si9233120wms.107 - gsmtp)
```

> Mostrar el correo recibido en gmail

![correo recibido](https://i.imgur.com/paotFxI.png)

Para estar seguros de que este correo ha llegado con SPF válido, tenemos que mostrar el correo original:

![checkeo pass válido](https://i.imgur.com/cUi9Kb7.png)

Ha pasado el checkeo SPF, pero si queremos ver información aún más específica, podemos hacer scroll al contenido en bruto del correo recibido:

![spf original](https://i.imgur.com/KSzFK0e.png)

Al ver eso, queda totalmente claro que el checkeo SPF ha sido exitoso.

### Tarea 2

> Crear registros MX y CNAME correspondiente en el DNS

![registros dns mx y mail](https://i.imgur.com/1pcAAMz.png)

> Habilitar en el firewall del vps el tráfico entrante al puerto 25

![firewall 25 vps](https://i.imgur.com/r6VLM8T.png)

> Enviar un correo desde gmail a la vps

![correo para vps](https://i.imgur.com/3duHQSd.png)

> Comprobar que la vps ha recibido correctamente el correo mirando `/var/log/mail.log`

![mail log vps](https://i.imgur.com/zfKjEvb.png)

> Muestro el correo recibido

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

### Crear tarea cron

Abro la configuración con:

```console
crontab -e
```

Añado lo siguiente:

```console
MAILTO = root

* * * * * echo Prueba de mensajes crontab
```

Esta tarea ejecuta un echo cada minuto y envía un correo a root con el resultado. No vemos ese echo por terminal ni en la sesión de nuestro usuario ni en la de root, pero nos sirve para hacer pruebas.

### Comprobar que llegan correos en root

```console
root@kampe:~# mail
"/var/mail/root": 64 messages 64 unread
>U   1 Cron Daemon        Mon Feb 14 21:20  23/788   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
 U   2 Cron Daemon        Mon Feb 14 21:21  22/762   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
 U   3 Cron Daemon        Mon Feb 14 21:22  22/762   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
 U   4 Cron Daemon        Mon Feb 14 21:23  22/762   Cron <blackmamba@kampe> echo Prueba de mensajes crontab
```

Como podemos ver, están llegando. Son todos iguales, así que muestro uno de ellos:

![correo crontab root](https://i.imgur.com/dWPZ8k3.png)

La parte que nos interesa es la marcada, el "From" y el "To". Los correos los envía el daemon de Cron usando root, hacia el usuario root.  
En el mensaje del correo está el resultado del echo.

Según como está ahora configurado cron, no nos llegarían esos correos a nuestro usuario.

### Crea un nuevo alias para que se manden a un usuario sin privilegios. 

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

### Comprobar que llegan a ese usuario

Ya están llegando:

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

He mostrado el contenido para que se vea el campo "To:", que apunta a root, pero el alias funciona así que me llega a mi usuario principal "blackmamba".

### Por último crea una redirección para enviar esos correo a gmail

Creo el fichero `/home/blackmamba/.forward` con el siguiente contenido:

```console
adristudy@gmail.com
```

Veo en gmail que me están llegando:

![correos cron a gmail](https://i.imgur.com/CV4QGD4.png)

## Gestión de correos desde un cliente

### Tarea 8

> Borrar el fichero `.forward` anteriormente creado

```console
rm .forward
```

Es **obligatorio** que borremos este fichero para que lo que hagamos durante esta tarea funcione, y los correos lleguen a `Maildir` correctamente.

¿Por qué? El fichero `.forward` tiene el siguiente funcionamiento:

![funcionamiento forward](https://i.imgur.com/pxhEqFg.png)

Es decir, **si tenemos un fichero `.forward` no podremos recibir ningún correo en usuarios locales**.

> Configurar postfix con buzón Maildir

Añado lo siguiente a `/etc/postfix/main.cf`:

```console
home_mailbox = Maildir/
```

Reinicio postfix:

```console
sudo systemctl restart postfix
```

> Enviar un correo a tu usuario y comprobar que el correo se ha guardado en el buzón Maildir. Recuerda que ese tipo de buzón no se puede leer con la utilidad mail.

Me envío un correo a mí mismo para hacer la prueba:

```console
blackmamba@kampe:~$ mail -r blackmamba@localhost blackmamba@localhost
Cc:
Subject: Prueba Maildir
Mensaje

```

Compruebo `/var/log/mail.log` para ver que haya llegado correctamente:

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



