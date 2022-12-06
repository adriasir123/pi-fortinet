# Ejercicios correo en escenario libvirt

## Ejercicio 1: Usuarios del mismo servidor

### Parte 1
> Instalar `postfix` en apolo

```
sudo apt install postfix
```

Selecciono "Internet Site":

![](https://i.postimg.cc/mg8J9xT5/internet-site-postfix.png)

Escribo mi nombre de dominio. Se tomará como dominio origen de los correos que se envíen desde apolo:

![](https://i.postimg.cc/T39T1Hyk/nombre-dominio-postfix.png)



### Parte 2
> Instalar `mailutils`

```
sudo apt install mailutils
```



### Parte 3
> Usar `mail` para enviar un correo desde el usuario `vagrant` al usuario `profesor`

```
vagrant@apolo:~$ mail profesor@localhost
Cc:
Subject: Correo para profesor
Correo de prueba

```
*(para que este correo se envíe, es necesario estar en una nueva línea del cuerpo y pulsar Ctrl+D)*



### Parte 4
> Leer el correo enviado usando `mail` en el usuario `profesor`

```
profesor@apolo:~$ mail
"/var/mail/profesor": 1 message 1 new
>N   1 vagrant@apolo.adri Tue Jan 18 09:38  13/528   Correo para profesor
? 1
Return-Path: <vagrant@apolo.adrianj.gonzalonazareno.org>
X-Original-To: profesor@localhost
Delivered-To: profesor@localhost
Received: by apolo.adrianj.gonzalonazareno.org (Postfix, from userid 1000)
	id C5521C02CA; Tue, 18 Jan 2022 09:38:24 +0000 (UTC)
To: <profesor@localhost>
Subject: Correo para profesor
X-Mailer: mail (GNU Mailutils 3.10)
Message-Id: <20220118093824.C5521C02CA@apolo.adrianj.gonzalonazareno.org>
Date: Tue, 18 Jan 2022 09:38:24 +0000 (UTC)
From: vagrant@apolo.adrianj.gonzalonazareno.org

Correo de prueba
?
```






## Ejercicio 2: Usuarios del servidor a correos de internet

### Parte 1
> Añadir `babuino-smtp.gonzalonazareno.org` como relayhost en postfix

Modifico `/etc/postfix/main.cf`:
```
relayhost = babuino-smtp.gonzalonazareno.org
```

Modifico los `nameserver` en apolo de la siguiente manera en `/etc/dhcp/dhclient.conf`:
```
supersede domain-name-servers 192.168.202.2,127.0.0.1;
```
*(he colocado la ip de `papion-dns` en primer orden para que al hacer relay se resuelva `babuino-smtp.gonzalonazareno.org` desde nuestra red y no desde Internet. Esto sucede porque apolo no tiene la zona `gonzalonazareno.org` y por ende acaba preguntando recursivamente)*



### Parte 2
> Enviar un correo a adristudy@gmail.com con `mail`

```
vagrant@apolo:~$ mail -r vagrant@adrianj.gonzalonazareno.org adristudy@gmail.com
Cc:
Subject: Correo externo
Prueba de correo a gmail

```
*(es necesario añadir con `-r` la dirección de origen. Así evitamos que `mail` use el hostname completo de la máquina como dominio de origen, lo cual sería incorrecto)*



### Parte 3
> Mostrar `/var/log/mail.log` para verificar que el correo se ha enviado correctamente

Muestro la línea relevante:
```
Jan 19 09:21:16 apolo postfix/smtp[2764]: 3EA6BC0D9E: to=<adristudy@gmail.com>, relay=babuino-smtp.gonzalonazareno.org[192.168.203.3]:25, delay=0.16, delays=0.06/0.03/0.06/0.01, dsn=2.0.0, status=sent (250 2.0.0 Ok: queued as 571FBFF778)
```



### Parte 4
> Mostrar el correo en gmail

![](https://i.postimg.cc/rmt7y2cd/correo-enviado-gmail.png)



### Parte 5
> ¿Cómo podemos ver el correo original (en bruto)?

![](https://i.imgur.com/u7He26f.png)



### Parte 6
> Indicar en las cabeceras la ruta de servidores SMTP tomada por el correo desde apolo a gmail

![](https://i.postimg.cc/rpwvsHcC/ruta-smpts-a-gmail.png)

*(los apartados "Received" se leen inversamente)*






## Ejercicio 3: correos desde internet a usuarios del servidor

### Parte 1
> Añadir registro MX a la vista externa de bind

Modifico `db.externa.adrianj.gonzalonazareno.org`:
```
$ORIGIN adrianj.gonzalonazareno.org.
$TTL 86400
@     IN     SOA    zeus.adrianj.gonzalonazareno.org.     admin.adrianj.gonzalonazareno.org. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

@              IN    NS      zeus.adrianj.gonzalonazareno.org.
@              IN    MX      10 zeus.adrianj.gonzalonazareno.org.

zeus           IN    A       172.22.0.213 ; clase
; zeus           IN    A       192.168.1.106 ; casa-eth
www            IN    CNAME   zeus
python         IN    CNAME   zeus
```

Reinicio bind
```
sudo systemctl restart bind9
```



### Parte 2
> DNAT en zeus redirigiendo el puerto 25 a apolo

```
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 25 -j DNAT --to-destination 10.0.1.102:25
```

Guardo la regla:
```
iptables-save > /etc/iptables/rules.v4
```



### Parte 3
> Responder desde gmail al correo recibido en el ejercicio anterior

Muestro mi respuesta:

![](https://i.postimg.cc/GpghS5k7/respuesta-desde-gmail.png)



### Parte 4
> Comprobar en apolo que se ha recibido correctamente el correo mirando `/var/log/mail.log`

```
Jan 19 09:55:10 apolo postfix/smtpd[2896]: connect from babuino-smtp.gonzalonazareno.org[192.168.203.3]
Jan 19 09:55:10 apolo postfix/smtpd[2896]: CACBDC0D97: client=babuino-smtp.gonzalonazareno.org[192.168.203.3]
Jan 19 09:55:10 apolo postfix/cleanup[2900]: CACBDC0D97: message-id=<CAG5uPReLQZ4YoD=8YG+JrvE6SYUjseL0M-NyD83KNoGHHbmzEw@mail.gmail.com>
Jan 19 09:55:10 apolo postfix/qmgr[2654]: CACBDC0D97: from=<adristudy@gmail.com>, size=3692, nrcpt=1 (queue active)
Jan 19 09:55:10 apolo postfix/smtpd[2896]: disconnect from babuino-smtp.gonzalonazareno.org[192.168.203.3] ehlo=2 starttls=1 mail=1 rcpt=1 data=1 quit=1 commands=7
Jan 19 09:55:10 apolo postfix/local[2901]: CACBDC0D97: to=<vagrant@adrianj.gonzalonazareno.org>, relay=local, delay=0.03, delays=0.02/0.01/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
Jan 19 09:55:10 apolo postfix/qmgr[2654]: CACBDC0D97: removed
```

Muestro el correo recibido:
```
vagrant@apolo:~$ mail
"/var/mail/vagrant": 5 messages 5 new
>N   1 adristudy          Wed Jan 19 09:41  72/3840  Re: Correo externo
 N   2 adristudy          Wed Jan 19 09:42  70/3685  Re: Correo externo
 N   3 adristudy          Wed Jan 19 09:42  71/3760  Re: Correo externo
 N   4 adristudy          Wed Jan 19 09:47  70/3733  Re: Correo externo
 N   5 adristudy          Wed Jan 19 09:55  71/3760  Re: Correo externo
? 5
Return-Path: <adristudy@gmail.com>
X-Original-To: vagrant@adrianj.gonzalonazareno.org
Delivered-To: vagrant@adrianj.gonzalonazareno.org
Received: from babuino-smtp.gonzalonazareno.org (babuino-smtp.gonzalonazareno.org [192.168.203.3])
	by apolo.adrianj.gonzalonazareno.org (Postfix) with ESMTPS id CACBDC0D97
	for <vagrant@adrianj.gonzalonazareno.org>; Wed, 19 Jan 2022 09:55:10 +0000 (UTC)
Received: from mail-pf1-f179.google.com (mail-pf1-f179.google.com [209.85.210.179])
	by babuino-smtp.gonzalonazareno.org (Postfix) with ESMTPS id A2EDAFF779
	for <vagrant@adrianj.gonzalonazareno.org>; Wed, 19 Jan 2022 09:55:10 +0000 (UTC)
Received: by mail-pf1-f179.google.com with SMTP id i17so2054870pfk.11
        for <vagrant@adrianj.gonzalonazareno.org>; Wed, 19 Jan 2022 01:55:10 -0800 (PST)
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=gmail.com; s=20210112;
        h=mime-version:references:in-reply-to:from:date:message-id:subject:to;
        bh=UBRzoQFstQM7RXR6pMZdQeQN69KQeIge6p3HzfH8Cnw=;
        b=SBed1/6ifJBmc3cotOM2KN5vYhx3Emlvd0snQssj4+19q2o0LEwG4yrLoDdmdoGCGk
         7akg1xd+ldFO9NFUDRRra5c7y/P0rZ/lOX8TILS7HRYVNnOtn3ko1e540sKhXYFzLeTR
         MBTBO0f/ru8+6zgQEMRHixsA7FGJldQLLv7TgJEzWSAFORcLg9VmHouXTH5e9+H/44E0
         zmkZI7WKS8vcjkVCaCt6Dfh/cjzI2NmQeMXGZzjVN30pLzu4YqcWyPjtjUv30SXSloLt
         Uz94rZ7Regq/geNg5GyzsYHWBuRINEjDHHbdN7TI9oOWEOt852gu2OgPLVw7Ls+8htyH
         gktA==
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=1e100.net; s=20210112;
        h=x-gm-message-state:mime-version:references:in-reply-to:from:date
         :message-id:subject:to;
        bh=UBRzoQFstQM7RXR6pMZdQeQN69KQeIge6p3HzfH8Cnw=;
        b=32Kq/nf+UvzvxQJ6ik5lUXbV7vsSA/vYiuKrf3UMopF8/xpmUziglRPbV6YKT77Klj
         00L/lqWR03204+B/rMc2fJuZgvL9mF1K7bhU5tUajx7K681EUyEfsb3SoHtFo5rACvpN
         faB5tspK0JGZpJrzlje+kOn2cMTTmA0fmbG9H1ZaZpbHqS8I88gUe5hTFd6g9b8Duo6D
         sLhcEt/PDaqpklT20i1dYq9PliJwrfV4w8Fphd1QB8JX6Q+LrfhrD59aLYNxgEXzb4+D
         le8cMo/T9FBRm1CGwOI4rikgOU7I7gAGk3lfdj7kz2z4WsgZbzNnhi10011eOzZkfWlJ
         CUSA==
X-Gm-Message-State: AOAM532fkZWJWnHiaF36A5Nc+fDMpsESW9z6MSRbA510wCRAVfpBggip
	Q1wcux1gIM2rqaJXsGRIiqgfjOexLLwBjtgCxx9vw/Rd2Ic=
X-Google-Smtp-Source: ABdhPJwFZl8S+5mmB9d4DO93QwZgmwFk0vh06sVosKBDr2J4J3PVhfQWB1g22JvwPa0VNnRtPizZaX37r8W8+b/Q3Yc=
X-Received: by 2002:a63:7c10:: with SMTP id x16mr26938561pgc.128.1642586106377;
 Wed, 19 Jan 2022 01:55:06 -0800 (PST)
MIME-Version: 1.0
References: <20220119092116.3EA6BC0D9E@apolo.adrianj.gonzalonazareno.org>
In-Reply-To: <20220119092116.3EA6BC0D9E@apolo.adrianj.gonzalonazareno.org>
From: adristudy <adristudy@gmail.com>
Date: Wed, 19 Jan 2022 10:54:55 +0100
Message-ID: <CAG5uPReLQZ4YoD=8YG+JrvE6SYUjseL0M-NyD83KNoGHHbmzEw@mail.gmail.com>
Subject: Re: Correo externo
To: vagrant@adrianj.gonzalonazareno.org
Content-Type: multipart/alternative; boundary="0000000000006238fe05d5ec61c4"

--0000000000006238fe05d5ec61c4
Content-Type: text/plain; charset="UTF-8"

respuesta para apolo

On Wed, 19 Jan 2022 at 10:21, <vagrant@adrianj.gonzalonazareno.org> wrote:

> Prueba de correo a gmail
>

--0000000000006238fe05d5ec61c4
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<div dir=3D"ltr"><div dir=3D"ltr"><div dir=3D"ltr">respuesta para apolo</di=
v></div></div><br><div class=3D"gmail_quote"><div dir=3D"ltr" class=3D"gmai=
l_attr">On Wed, 19 Jan 2022 at 10:21, &lt;<a href=3D"mailto:vagrant@adrianj=
.gonzalonazareno.org">vagrant@adrianj.gonzalonazareno.org</a>&gt; wrote:<br=
></div><blockquote class=3D"gmail_quote" style=3D"margin:0px 0px 0px 0.8ex;=
border-left:1px solid rgb(204,204,204);padding-left:1ex">Prueba de correo a=
 gmail<br>
</blockquote></div>

--0000000000006238fe05d5ec61c4--
```
*(escribo 5 porque es el último recibido)*
