# Práctica: Integridad, firmas y autenticación

## Tarea 1: Firmas electrónicas

### 1.1

> Enviar un fichero y la firma del mismo a Carlos

Enviaré:

```console
-rw-r--r-- 1 vagrant vagrant 28 Apr 28 01:03 paracarlos.txt
```

Genero la firma:

```console
gpg --detach-sign paracarlos.txt
```

Obtengo:

```console
-rw-r--r-- 1 vagrant vagrant       438 Apr 28 01:12 paracarlos.txt.sig
```

Esta firma también la enviaré.  
Ambos ficheros enviados por scp.

> Verificar la firma recibida

Recibo por scp:

```console
-rw-r--r-- 1 vagrant vagrant        28 May 15 11:02 paraadrian.txt
-rw-r--r-- 1 vagrant vagrant       438 May 15 11:02 paraadrian.txt.sig
```

Verifico la firma:

```console
vagrant@practicafirmas:~$ gpg --verify paraadrian.txt.sig paraadrian.txt
gpg: Signature made Sun May 15 10:44:25 2022 UTC
gpg:                using RSA key 4E8FF10EDF312F43014B7AE13F6189C8E59B7714
gpg: Good signature from "Carlos Rivero <carlosrivero198@gmail.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 4E8F F10E DF31 2F43 014B  7AE1 3F61 89C8 E59B 7714
```

La verificación ha sido correcta.

### 1.2

> ¿Qué significa este mensaje al verificar la firma?

```console
 gpg: Firma correcta de "Pepe D <josedom24@gmail.com>" [desconocido]
 gpg: ATENCIÓN: ¡Esta clave no está certificada por una firma de confianza!
 gpg:          No hay indicios de que la firma pertenezca al propietario.
 Huellas dactilares de la clave primaria: E8DD 5DA9 3B88 F08A DA1D  26BF 5141 3DDB 0C99 55FC
```

Significa que la clave pública con la que se ha verificado la firma, no es una clave en la que yo confíe / haya firmado.

Por ejemplo, puede suceder cuando la trust es "unknown", como ha sido el caso del apartado anterior.

### 1.3

En este apartado crearemos un anillo de confianza entre varios compañeros.

#### 1.3.1

> Subir tu clave pública a `pgp.rediris.es`

```console
vagrant@practicafirmas:~$ gpg --keyserver pgp.rediris.es --send-keys 69172731E9645F6ACB60CCBA0C3C966CD1DC1A62
gpg: sending key 0C3C966CD1DC1A62 to hkp://pgp.rediris.es
```

Compruebo que se ha subido correctamente:

```console
vagrant@practicafirmas:~$ gpg --keyserver pgp.rediris.es --search-keys 69172731E9645F6ACB60CCBA0C3C966CD1DC1A62
gpg: data source: http://130.206.1.8:11371
(1)	Adrian Jaramillo <adristudy@gmail.com>
	  3072 bit RSA key 0C3C966CD1DC1A62, created: 2022-05-16, expires: 2024-05-15
Keys 1-1 of 1 for "69172731E9645F6ACB60CCBA0C3C966CD1DC1A62".  Enter number(s), N)ext, or Q)uit >
```

#### 1.3.2

> Pasar el fingerprint de tu clave subida a tus compañeros para que la puedan descargar

Hecho.

#### 1.3.3

> Bajar y firmar tres claves públicas de compañeros

Me bajo la de Carlos:

```console
vagrant@practicafirmas:~$ gpg --keyserver pgp.rediris.es --receive-keys 3F6189C8E59B7714
gpg: key 3F6189C8E59B7714: public key "Carlos Rivero <carlosrivero198@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

La firmo:

```console
vagrant@practicafirmas:~$ gpg --sign-key 3F6189C8E59B7714

pub  rsa3072/3F6189C8E59B7714
     created: 2022-05-15  expires: 2024-05-14  usage: SC
     trust: unknown       validity: unknown
sub  rsa3072/1707D1A73D4578B2
     created: 2022-05-15  expires: 2024-05-14  usage: E
[ unknown] (1). Carlos Rivero <carlosrivero198@gmail.com>


pub  rsa3072/3F6189C8E59B7714
     created: 2022-05-15  expires: 2024-05-14  usage: SC
     trust: unknown       validity: unknown
 Primary key fingerprint: 4E8F F10E DF31 2F43 014B  7AE1 3F61 89C8 E59B 7714

     Carlos Rivero <carlosrivero198@gmail.com>

This key is due to expire on 2024-05-14.
Are you sure that you want to sign this key with your
key "Adrian Jaramillo <adristudy@gmail.com>" (0C3C966CD1DC1A62)

Really sign? (y/N) y
```

Muestro que se aplicó correctamente:

```console
vagrant@practicafirmas:~$ gpg --list-sig 4E8FF10EDF312F43014B7AE13F6189C8E59B7714
pub   rsa3072 2022-05-15 [SC] [expires: 2024-05-14]
      4E8FF10EDF312F43014B7AE13F6189C8E59B7714
uid           [  full  ] Carlos Rivero <carlosrivero198@gmail.com>
sig 3        3F6189C8E59B7714 2022-05-15  Carlos Rivero <carlosrivero198@gmail.com>
sig          0C3C966CD1DC1A62 2022-05-16  Adrian Jaramillo <adristudy@gmail.com>
sub   rsa3072 2022-05-15 [E] [expires: 2024-05-14]
sig          3F6189C8E59B7714 2022-05-15  Carlos Rivero <carlosrivero198@gmail.com>
```

Me bajo la de Daniel:

```console
vagrant@practicafirmas:~$ gpg --keyserver pgp.rediris.es --receive-keys 82805738F5FF4457
gpg: key 82805738F5FF4457: public key "Daniel Parrales <daniparrales1@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

La firmo:

```console
vagrant@practicafirmas:~$ gpg --sign-key 82805738F5FF4457

pub  rsa3072/82805738F5FF4457
     created: 2022-05-16  expires: 2024-05-15  usage: SC
     trust: unknown       validity: unknown
sub  rsa3072/408742D95A6657E5
     created: 2022-05-16  expires: 2024-05-15  usage: E
[ unknown] (1). Daniel Parrales <daniparrales1@gmail.com>


pub  rsa3072/82805738F5FF4457
     created: 2022-05-16  expires: 2024-05-15  usage: SC
     trust: unknown       validity: unknown
 Primary key fingerprint: 8595 30FB 6525 B22C 5F36  8117 8280 5738 F5FF 4457

     Daniel Parrales <daniparrales1@gmail.com>

This key is due to expire on 2024-05-15.
Are you sure that you want to sign this key with your
key "Adrian Jaramillo <adristudy@gmail.com>" (0C3C966CD1DC1A62)

Really sign? (y/N) y
```

Muestro que se aplicó correctamente:

```console
vagrant@practicafirmas:~$ gpg --list-sig 82805738F5FF4457
pub   rsa3072 2022-05-16 [SC] [expires: 2024-05-15]
      859530FB6525B22C5F36811782805738F5FF4457
uid           [  full  ] Daniel Parrales <daniparrales1@gmail.com>
sig 3        82805738F5FF4457 2022-05-16  Daniel Parrales <daniparrales1@gmail.com>
sig          0C3C966CD1DC1A62 2022-06-07  Adrian Jaramillo <adristudy@gmail.com>
sub   rsa3072 2022-05-16 [E] [expires: 2024-05-15]
sig          82805738F5FF4457 2022-05-16  Daniel Parrales <daniparrales1@gmail.com>
```

Me bajo la de Lara:

```console
vagrant@practicafirmas:~$ gpg --keyserver pgp.rediris.es --receive-keys EB5FEF455BD986A0
gpg: key EB5FEF455BD986A0: public key "Lara Pruna Ternero <laraprute@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

La firmo:

```console
vagrant@practicafirmas:~$ gpg --sign-key EB5FEF455BD986A0

pub  rsa3072/EB5FEF455BD986A0
     created: 2022-06-07  expires: 2024-06-06  usage: SC
     trust: unknown       validity: unknown
sub  rsa3072/C1F92F5A0AEA093D
     created: 2022-06-07  expires: 2024-06-06  usage: E
[ unknown] (1). Lara Pruna Ternero <laraprute@gmail.com>


pub  rsa3072/EB5FEF455BD986A0
     created: 2022-06-07  expires: 2024-06-06  usage: SC
     trust: unknown       validity: unknown
 Primary key fingerprint: 8317 2CA2 F8FC 2124 94CB  677D EB5F EF45 5BD9 86A0

     Lara Pruna Ternero <laraprute@gmail.com>

This key is due to expire on 2024-06-06.
Are you sure that you want to sign this key with your
key "Adrian Jaramillo <adristudy@gmail.com>" (0C3C966CD1DC1A62)

Really sign? (y/N) y
```

Muestro que se aplicó correctamente:

```console
vagrant@practicafirmas:~$ gpg --list-sig EB5FEF455BD986A0
pub   rsa3072 2022-06-07 [SC] [expires: 2024-06-06]
      83172CA2F8FC212494CB677DEB5FEF455BD986A0
uid           [  full  ] Lara Pruna Ternero <laraprute@gmail.com>
sig 3        EB5FEF455BD986A0 2022-06-07  Lara Pruna Ternero <laraprute@gmail.com>
sig          0C3C966CD1DC1A62 2022-06-07  Adrian Jaramillo <adristudy@gmail.com>
sub   rsa3072 2022-06-07 [E] [expires: 2024-06-06]
sig          EB5FEF455BD986A0 2022-06-07  Lara Pruna Ternero <laraprute@gmail.com>
```

#### 1.3.4

> Tu clave pública tiene que tener 3 firmas

```console
vagrant@practicafirmas:~$ gpg --list-sigs adristudy@gmail.com
pub   rsa3072 2022-05-16 [SC] [expires: 2024-05-15]
      69172731E9645F6ACB60CCBA0C3C966CD1DC1A62
uid           [ultimate] Adrian Jaramillo <adristudy@gmail.com>
sig 3        0C3C966CD1DC1A62 2022-05-16  Adrian Jaramillo <adristudy@gmail.com>
sig          3F6189C8E59B7714 2022-06-07  Carlos Rivero <carlosrivero198@gmail.com>
sig          82805738F5FF4457 2022-06-07  Daniel Parrales <daniparrales1@gmail.com>
sig          EB5FEF455BD986A0 2022-06-07  Lara Pruna Ternero <laraprute@gmail.com>
sub   rsa3072 2022-05-16 [E] [expires: 2024-05-15]
sig          0C3C966CD1DC1A62 2022-05-16  Adrian Jaramillo <adristudy@gmail.com>
```

#### 1.3.5

> Al firmar claves, devolverlas a su dueño

Todas las exporto de esta manera:

```console
gpg --export -a carlosrivero198@gmail.com > publiccarlos.key
```

#### 1.3.6

> Subir a `pgp.rediris.es` tu clave cuando tenga tres firmas

```console
vagrant@practicafirmas:~$ gpg --keyserver pgp.rediris.es --send-key 69172731E9645F6ACB60CCBA0C3C966CD1DC1A62
gpg: sending key 0C3C966CD1DC1A62 to hkp://pgp.rediris.es
```

Muestro las 3 firmas en el keyserver:

![miclaveconfirmas](https://i.postimg.cc/rp2cHB9p/3-firmas-miclave.png)

> Rellenar datos de tu clave pública [aquí](https://dit.gonzalonazareno.org/redmine/projects/asir2/wiki/Claves_p%C3%BAblicas_PGP_2021-2022)

![miclavedeclase](https://i.imgur.com/ceSgH9c.png)

#### 1.3.7

> Bajar las claves públicas de tus compañeros con tres firmas

Todas se bajarán de la misma manera:

```console
gpg --keyserver pgp.rediris.es --recv-key 3F6189C8E59B7714
```

**IMPORTANTE: AL BAJAR DIRECTAMENTE LAS CLAVES DE ESTA MANERA LAS FIRMAS SE ELIMINAN, ASÍ QUE ES MEJOR IMPORTAR LAS CLAVES DESDE UN FICHERO Y ASÍ MANTENDREMOS LAS FIRMAS**

### 1.4

> Mostrar las firmas de tu clave pública

```console
vagrant@practicafirmas:~$ gpg --list-sigs adristudy@gmail.com
pub   rsa3072 2022-05-16 [SC] [expires: 2024-05-15]
      69172731E9645F6ACB60CCBA0C3C966CD1DC1A62
uid           [ultimate] Adrian Jaramillo <adristudy@gmail.com>
sig 3        0C3C966CD1DC1A62 2022-05-16  Adrian Jaramillo <adristudy@gmail.com>
sig          3F6189C8E59B7714 2022-06-07  Carlos Rivero <carlosrivero198@gmail.com>
sig          82805738F5FF4457 2022-06-07  Daniel Parrales <daniparrales1@gmail.com>
sig          EB5FEF455BD986A0 2022-06-07  Lara Pruna Ternero <laraprute@gmail.com>
sub   rsa3072 2022-05-16 [E] [expires: 2024-05-15]
sig          0C3C966CD1DC1A62 2022-05-16  Adrian Jaramillo <adristudy@gmail.com>
```

### 1.5

> Comprobar que se puede verificar "sin problemas” la firma de Carlos, porque ya se confía

```console
vagrant@practicafirmas:~$ gpg --verify paraadrian.txt.sig paraadrian.txt
gpg: Signature made Sun May 15 10:44:25 2022 UTC
gpg:                using RSA key 4E8FF10EDF312F43014B7AE13F6189C8E59B7714
gpg: Good signature from "Carlos Rivero <carlosrivero198@gmail.com>" [full]
```

### 1.6

> Comprobar que puedes verificar con confianza una firma de una persona en la que no confías, pero sin embargo si confía otra persona en la que tu tienes confianza total

En este apartado vamos a intentar verificar una firma de Miguel:

```console
-rw-r--r-- 1 vagrant vagrant        16 Jun  8 06:17 paraadriandemiguel.txt
-rw-r--r-- 1 vagrant vagrant       438 Jun  8 06:18 paraadriandemiguel.txt.sig
```

Tenemos su clave pública firmada por Carlos (en el que confiamos), pero la propia clave de Miguel nosotros no la hemos firmado, por lo que no confiamos:

```console
vagrant@practicafirmas:~$ gpg --list-sigs miguelcor.rrs@gmail.com
pub   rsa3072 2022-06-07 [SC] [expires: 2024-06-06]
      5173B9123875E25C833A855307F879A05A4147C3
uid           [  undef ] Miguel Cordoba <miguelcor.rrs@gmail.com>
sig 3        07F879A05A4147C3 2022-06-07  Miguel Cordoba <miguelcor.rrs@gmail.com>
sig          3F6189C8E59B7714 2022-06-07  Carlos Rivero <carlosrivero198@gmail.com>
sub   rsa3072 2022-06-07 [E] [expires: 2024-06-06]
sig          07F879A05A4147C3 2022-06-07  Miguel Cordoba <miguelcor.rrs@gmail.com>
```

Bien, si intentamos verificar la firma de Miguel, ¿debería de no haber errores verdad? Porque como dijimos, Carlos firmó la de Miguel, y yo confío en Carlos.

Hagamos la prueba:

```console
vagrant@practicafirmas:~$ gpg --verify paraadriandemiguel.txt.sig paraadriandemiguel.txt
gpg: Signature made Wed Jun  8 06:15:58 2022 UTC
gpg:                using RSA key 5173B9123875E25C833A855307F879A05A4147C3
gpg: Good signature from "Miguel Cordoba <miguelcor.rrs@gmail.com>" [undefined]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 5173 B912 3875 E25C 833A  8553 07F8 79A0 5A41 47C3
```

Error. ¿Por qué ha sucedido esto?

Pues bien, debemos explicar antes 2 conceptos fundamentales, **trust y validity**:

* **TRUST**: la trust se configura manualmente. Describe si una clave puede firmar otras claves, y que se confíen en ellas aunque yo directamente no confíe (permite la confianza en terceros). Ej: Si se confía completamente en una persona "A", y esa persona firma la clave de una persona "B", automáticamente veremos la clave de "B" como válida.
* **VALIDITY**: la validez en diferencia con la trust, sí puede ser automática. Describe si una clave tiene las suficientes firmas para que se pruebe que es la clave correcta (cuando nosotros firmamos una clave, automáticamente la validez pasa a "full")

Habiendo dicho esto, entonces lo que ha debido pasar es que no tenemos confianza en la clave de Carlos, aunque la validez esté a full porque la firmamos en su momento. Es decir, sabemos que la clave de Carlos es de verdad de Carlos, pero no confiamos en que Carlos firme otras claves, y luego nos creamos que esas otras son verdaderas.

Vamos a mostrar los datos de la clave de Carlos para comprobar lo ya dicho:

```console
vagrant@practicafirmas:~$ gpg --edit-key carlosrivero198@gmail.com
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  rsa3072/3F6189C8E59B7714
     created: 2022-05-15  expires: 2024-05-14  usage: SC
     trust: unknown       validity: full
sub  rsa3072/1707D1A73D4578B2
     created: 2022-05-15  expires: 2024-05-14  usage: E
[  full  ] (1). Carlos Rivero <carlosrivero198@gmail.com>

gpg>
```

Está ocurriendo tal y como hemos explicado antes, la trust es "unknown" y la validity es "full". No estamos confiando en las firmas a terceros de Carlos.

¿Entonces qué debemos hacer?

Cambiar la trust en la clave de Carlos para que confiemos en sus firmas a terceros. Debemos configurar el nivel de trust a 4, o full. Lo hacemos así:

```console
vagrant@practicafirmas:~$ gpg --edit-key 4E8FF10EDF312F43014B7AE13F6189C8E59B7714 trust
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  rsa3072/3F6189C8E59B7714
     created: 2022-05-15  expires: 2024-05-14  usage: SC
     trust: unknown       validity: full
sub  rsa3072/1707D1A73D4578B2
     created: 2022-05-15  expires: 2024-05-14  usage: E
[  full  ] (1). Carlos Rivero <carlosrivero198@gmail.com>

pub  rsa3072/3F6189C8E59B7714
     created: 2022-05-15  expires: 2024-05-14  usage: SC
     trust: unknown       validity: full
sub  rsa3072/1707D1A73D4578B2
     created: 2022-05-15  expires: 2024-05-14  usage: E
[  full  ] (1). Carlos Rivero <carlosrivero198@gmail.com>

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 4

pub  rsa3072/3F6189C8E59B7714
     created: 2022-05-15  expires: 2024-05-14  usage: SC
     trust: full          validity: full
sub  rsa3072/1707D1A73D4578B2
     created: 2022-05-15  expires: 2024-05-14  usage: E
[  full  ] (1). Carlos Rivero <carlosrivero198@gmail.com>
Please note that the shown key validity is not necessarily correct
unless you restart the program.

gpg> quit
```

Ahora la trust está a full como podemos observar.

Pues bien, ahora sí podremos verificar correctamente la firma de Miguel, porque ahora confiamos en las firmas de Carlos a terceros. Por esto, en nuestro listado de claves, ahora la clave de Miguel es confiada de forma full:

```console
pub   rsa3072 2022-06-07 [SC] [expires: 2024-06-06]
      5173B9123875E25C833A855307F879A05A4147C3
uid           [  full  ] Miguel Cordoba <miguelcor.rrs@gmail.com>
sub   rsa3072 2022-06-07 [E] [expires: 2024-06-06]
```

Podremos ahora verificar su firma correctamente:

```console
vagrant@practicafirmas:~$ gpg --verify paraadriandemiguel.txt.sig paraadriandemiguel.txt
gpg: Signature made Wed Jun  8 06:15:58 2022 UTC
gpg:                using RSA key 5173B9123875E25C833A855307F879A05A4147C3
gpg: Good signature from "Miguel Cordoba <miguelcor.rrs@gmail.com>" [full]
```

## Tarea 2: Correo seguro con Evolution

### 2.1

> Configurar una cuenta de correo

```console
sudo apt install evolution
```

Después de instalarlo y configurarlo, muestro que funciona:

![evolution funcionando](https://i.imgur.com/xdvQd42.png)

### 2.2

> Configurar el firmado de correos con nuestra clave privada

Primero tenemos que llegar al menú correspondiente:

![evolution config firmas](https://i.postimg.cc/prBYF7b5/llegar-config-firmado-evolution.gif)

Una vez ahí, indicamos el fingerprint del par de claves con el que queremos firmar:

![config evolution firmado](https://i.imgur.com/1owmbRJ.png)

Para que los correos de verdad se firmen al enviarse, tenemos que marcar la siguiente opción antes de enviarlos:

![marcar correos para firmado](https://i.imgur.com/sQ7B3iT.png)

### 2.3

> Configurar el cifrado de correos para otros destinatarios

Para el cifrado no hace falta que configuremos en Evolution las claves públicas, ya que las toma de nuestro keybox automáticamente según la dirección que escribamos en el *"To:"*.

Para que los correos de verdad se cifren al enviarse, tenemos que marcar la siguiente opción antes de enviarlos:

![marcar correos para cifrado](https://i.imgur.com/rhWfSdx.png)

### 2.4

> Enviar un correo firmado y cifrado a Carlos

Este es el correo que enviaré:

![firmado/cifrado para carlos](https://i.imgur.com/Yxc5hAX.png)

Si yo lo intento abrir, no podré ver el contenido ya que no tengo la clave privada de Carlos para desencriptarlo:

![intento abrir correo para carlos](https://i.imgur.com/znoJDqT.png)

Este es el correo que le ha llegado a Carlos, desde su punto de vista, y vemos que todo es correcto:

![correo que carlos ha recibido](https://i.imgur.com/5Yq8DNm.png)

### 2.5

> Recibir un correo firmado y cifrado de Carlos

Vemos que todo es correcto:

![correo que he recibido de Carlos](https://i.imgur.com/sEaWbLf.png)

### 2.6

> Mandar a Raúl un correo cifrado y firmado

Busco la clave pública de Raúl en RedIRIS y me la bajo (se necesitará para encriptar):

```console
atlas@olympus:~$ gpg --keyserver pgp.rediris.es --search-key rruizp@gmail.com
gpg: data source: http://130.206.1.8:11371
(1)	Raúl Ruiz Padilla <rruizp@gmail.com>
	  3072 bit RSA key A036DF623D608DE0, created: 2021-11-10, expires: 2023-11-10
(2)	Raúl Ruiz (Claves para clase) <rruizp@gmail.com>
	  4096 bit RSA key 334BEE7E18D37DEA, created: 2017-12-16
(3)	Raul Ruiz Padilla (Clave para SAD 1617) <rruizp@gmail.com>
	  4096 bit RSA key F6C88DC57C2FA472, created: 2016-12-11, expires: 2018-12-10 (expired)
Keys 1-3 of 3 for "rruizp@gmail.com".  Enter number(s), N)ext, or Q)uit > 1
gpg: key A036DF623D608DE0: public key "Raúl Ruiz Padilla <rruizp@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Antes de enviar el correo es importante que dejemos marcada esta opción, para evitar el tener que firmar las claves públicas de las personas a quienes queramos enviar:

![confiar en claves al encriptar](https://i.imgur.com/zT4Rliu.png)

Este es el correo que enviaré:

![correo enviado a raúl](https://i.imgur.com/tNZWIHq.png)

La parte marcada en rojo quiere decir que el correo se encriptará y firmará.

**¿QUÉ TENDRÁ QUE HACER RAÚL PARA LEER ESTE CORREO CORRECTAMENTE?**

Necesitará 2 cosas:

* La clave privada correspondiente a la pública con la que se encriptó el correo para descifrarlo correctamente.
* Mi clave pública para poder verificar la firma. La podrá obtener con el comando `gpg --keyserver pgp.rediris.es --search-key adristudy@gmail.com`, eligiendo la de fingerprint CA350E4C295E2EF4.

## Tarea 3: Integridad de ficheros

### 3.1

> Descargar la ISO de Debian

```console
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-11.2.0-amd64-netinst.iso
```

Obtengo el fichero:

```console
-rw-r--r-- 1 vagrant vagrant 396361728 Dec 18 13:24 debian-11.2.0-amd64-netinst.iso
```

### 3.2

> Comprobar la integridad de la ISO descargada usando `sha512sum`

En el fichero [SHA512SUMS](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS) que nos proporciona Debian, nos encontramos el siguiente checksum para la ISO descargada:

```console
c685b85cf9f248633ba3cd2b9f9e781fa03225587e0c332aef2063f6877a1f0622f56d44cf0690087b0ca36883147ecb5593e3da6f965968402cdbdf12f6dd74
```

Luego utilizo `sha512sum` sobre la ISO que me he descargado:

```console
sha512sum debian-11.2.0-amd64-netinst.iso
```

```console
c685b85cf9f248633ba3cd2b9f9e781fa03225587e0c332aef2063f6877a1f0622f56d44cf0690087b0ca36883147ecb5593e3da6f965968402cdbdf12f6dd74  debian-11.2.0-amd64-netinst.iso
```

Ambos hashes coinciden, así que nos hemos descargado el fichero correctamente.

Igualmente, si queremos evitar errores, podemos usar [una herramienta online](https://text-compare.com/) para comparar strings:

![comparación de hashes](https://i.imgur.com/KUhapSW.png)

### 3.3

> Verificar que `SHA512SUMS` no ha sido manipulado usando `SHA512SUMS.sign`

Primero descargo ambos ficheros:

```console
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS.sign
```

Muestro que los tengo:

```console
-rw-r--r-- 1 vagrant vagrant       494 Dec 18 20:42 SHA512SUMS
-rw-r--r-- 1 vagrant vagrant       833 Dec 18 20:45 SHA512SUMS.sign
```

Averiguo el fingerprint del par de claves que se usó originalmente para crear la firma `SHA512SUMS.sign`:

```console
gpg --verify SHA512SUMS.sign
```

![averiguando fingerprint](https://i.imgur.com/pSmi6pt.png)

No se puede verificar la firma porque no tenemos la clave pública de Debian en nuestro keybox, pero ya sabemos su fingerprint para descargarla:

```console
gpg --keyserver keyring.debian.org --recv DF9B9C49EAA9298432589D76DA87E80D6294BE9B
```

```console
gpg: key DA87E80D6294BE9B: public key "Debian CD signing key <debian-cd@lists.debian.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Finalmente, podemos verificar la firma:

```console
gpg --verify SHA512SUMS.sign SHA512SUMS
```

```console
gpg: Signature made Sat Dec 18 20:45:36 2021 UTC
gpg:                using RSA key DF9B9C49EAA9298432589D76DA87E80D6294BE9B
gpg: Good signature from "Debian CD signing key <debian-cd@lists.debian.org>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: DF9B 9C49 EAA9 2984 3258  9D76 DA87 E80D 6294 BE9B
```

Se ha podido verificar correctamente, `SHA512SUMS` es legítimo.

## Tarea 4: Integridad y autenticidad (apt-secure)

### 4.1

> ¿Qué herramienta utiliza apt-secure para firmar y verificar firmas?

GPG.

### 4.2

> ¿Para qué sirve `apt-key`?

Se usa para gestionar la lista de claves que mantiene apt para autenticar paquetes.  

Los paquetes que sean autenticados usando estas claves se considerarán "trusted".

> ¿Qué muestra el comando `apt-key list`?

Lista las claves públicas en las que confiamos.  

Muestro mi output de `apt-key list`:

```console
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
/etc/apt/trusted.gpg.d/debian-archive-bullseye-automatic.gpg
------------------------------------------------------------
pub   rsa4096 2021-01-17 [SC] [expires: 2029-01-15]
      1F89 983E 0081 FDE0 18F3  CC96 73A4 F27B 8DD4 7936
uid           [ unknown] Debian Archive Automatic Signing Key (11/bullseye) <ftpmaster@debian.org>
sub   rsa4096 2021-01-17 [S] [expires: 2029-01-15]

/etc/apt/trusted.gpg.d/debian-archive-bullseye-security-automatic.gpg
---------------------------------------------------------------------
pub   rsa4096 2021-01-17 [SC] [expires: 2029-01-15]
      AC53 0D52 0F2F 3269 F5E9  8313 A484 4904 4AAD 5C5D
uid           [ unknown] Debian Security Archive Automatic Signing Key (11/bullseye) <ftpmaster@debian.org>
sub   rsa4096 2021-01-17 [S] [expires: 2029-01-15]

/etc/apt/trusted.gpg.d/debian-archive-bullseye-stable.gpg
---------------------------------------------------------
pub   rsa4096 2021-02-13 [SC] [expires: 2029-02-11]
      A428 5295 FC7B 1A81 6000  62A9 605C 66F0 0D6C 9793
uid           [ unknown] Debian Stable Release Key (11/bullseye) <debian-release@lists.debian.org>

/etc/apt/trusted.gpg.d/debian-archive-buster-automatic.gpg
----------------------------------------------------------
pub   rsa4096 2019-04-14 [SC] [expires: 2027-04-12]
      80D1 5823 B7FD 1561 F9F7  BCDD DC30 D7C2 3CBB ABEE
uid           [ unknown] Debian Archive Automatic Signing Key (10/buster) <ftpmaster@debian.org>
sub   rsa4096 2019-04-14 [S] [expires: 2027-04-12]

/etc/apt/trusted.gpg.d/debian-archive-buster-security-automatic.gpg
-------------------------------------------------------------------
pub   rsa4096 2019-04-14 [SC] [expires: 2027-04-12]
      5E61 B217 265D A980 7A23  C5FF 4DFA B270 CAA9 6DFA
uid           [ unknown] Debian Security Archive Automatic Signing Key (10/buster) <ftpmaster@debian.org>
sub   rsa4096 2019-04-14 [S] [expires: 2027-04-12]

/etc/apt/trusted.gpg.d/debian-archive-buster-stable.gpg
-------------------------------------------------------
pub   rsa4096 2019-02-05 [SC] [expires: 2027-02-03]
      6D33 866E DD8F FA41 C014  3AED DCC9 EFBF 77E1 1517
uid           [ unknown] Debian Stable Release Key (10/buster) <debian-release@lists.debian.org>

/etc/apt/trusted.gpg.d/debian-archive-stretch-automatic.gpg
-----------------------------------------------------------
pub   rsa4096 2017-05-22 [SC] [expires: 2025-05-20]
      E1CF 20DD FFE4 B89E 8026  58F1 E0B1 1894 F66A EC98
uid           [ unknown] Debian Archive Automatic Signing Key (9/stretch) <ftpmaster@debian.org>
sub   rsa4096 2017-05-22 [S] [expires: 2025-05-20]

/etc/apt/trusted.gpg.d/debian-archive-stretch-security-automatic.gpg
--------------------------------------------------------------------
pub   rsa4096 2017-05-22 [SC] [expires: 2025-05-20]
      6ED6 F5CB 5FA6 FB2F 460A  E88E EDA0 D238 8AE2 2BA9
uid           [ unknown] Debian Security Archive Automatic Signing Key (9/stretch) <ftpmaster@debian.org>
sub   rsa4096 2017-05-22 [S] [expires: 2025-05-20]

/etc/apt/trusted.gpg.d/debian-archive-stretch-stable.gpg
--------------------------------------------------------
pub   rsa4096 2017-05-20 [SC] [expires: 2025-05-18]
      067E 3C45 6BAE 240A CEE8  8F6F EF0F 382A 1A7B 6500
uid           [ unknown] Debian Stable Release Key (9/stretch) <debian-release@lists.debian.org>
```

### 4.3

> ¿Dónde se encuentran los distintos keyrings que usa `apt-key`?

En `/etc/apt/trusted.gpg.d/`.

### 4.4

> ¿Qué contiene el fichero `Release` de un repositorio?

En otros datos, contiene los checksums de los ficheros que hay en el repositorio.

Ejemplo: <http://ftp.debian.org/debian/dists/Debian11.2/Release>

> ¿Y `Release.gpg`?

Contiene la firma detached ascii-armored del fichero `Release`.

Ejemplo: <http://ftp.debian.org/debian/dists/Debian11.2/Release.gpg>

### 4.5

> Explicar el proceso por el cual apt nos asegura que los ficheros que estamos descargando son legítimos

1. Se descargan los ficheros `Release`, `Release.gpg`, y el fichero `Packages` que corresponda.
2. Se comprueba usando la firma `Release.gpg` más la clave pública correspondiente, que el fichero `Release` es legítimo. A partir de aquí, apt ya confía en todos sus checksums.
3. Apt compara el checksum del fichero `Packages` con el que encuentra en el fichero `Release`; si coinciden, quiere decir que se descargó el fichero correcto.
4. Al descargarnos un paquete individual se compara su checksum con el que aparece en `Packages`; si coinciden, quiere decir que se descargó el paquete correcto.

### 4.6

> Añadir el repositorio de VirtualBox

Añado las claves gpg necesarias:

```console
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
```

Añado el repositorio:

```console
echo "deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian bullseye contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list
```

## Tarea 5: Autenticación en SSH

### 5.1

> Explicar qué sucede entre el cliente y el servidor para que se cifre la comunicación

Una sesión SSH tiene 2 fases antes de que se pueda cifrar la comunicación. Las explicaré a continuación.

1. **CONEXIÓN**
    1. El cliente establece una conexión TCP con el servidor, y este responde con las versiones del protocolo que soporta.
    2. Si el cliente usa una versión del protocolo que coincida con las que acepta el servidor, la conexión continúa.
    3. El servidor comparte su clave pública con el cliente, para que este compruebe si efectivamente es la máquina a la que quería conectarse.
    4. A partir de ahora, ambas partes están preparadas para la negociación de una clave de sesión usando Diffie-Hellman.
2. **NEGOCIACIÓN DE LA ENCRIPTACIÓN PARA LA SESIÓN**
    1. Ambas partes se ponen de acuerdo en un número primo alto, que servirá como "seed value".
    2. Ambas partes se ponen de acuerdo en un tipo de cifrado (normalmente AES).
    3. Ambas partes generan otro número primo por separado, que mantienen en secreto de la otra parte. Se usará como clave privada durante esta fase de negociación *(esta clave privada NO es la misma que se usa para la autenticación)*.
    4. La clave privada generada + el tipo de cifrado + el número primo compartido, se usarán para generar una clave pública que estará derivada de la clave privada.
    5. Ambas partes intercambian sus claves públicas generadas *(estas claves públicas NO son las mismas que se usan para la autenticación)*.
    6. Cada parte usará su clave privada + la clave pública de la otra parte + el número primo compartido original, para calcular una clave compartida "shared secret". Aunque este proceso lo realiza cada parte por separado, mientras se usen claves públicas y privadas opuestas, se llegará al mismo "shared secret".
    7. Este "shared secret" se usará para encriptar toda comunicación a partir de ahora. Es una clave simétrica.

> ¿Para qué se utiliza la criptografía simétrica?

Para encriptar las conexiones, mediante el uso de una clave secreta.

Esta se genera a través de un proceso llamado "key exchange algorithm",
y es única por sesión.  
En cuanto ambas partes la sepan, el resto de la sesión se encriptará usando esta clave.

> ¿Y la asimétrica?

Se usa:

* Durante el intercambio de clave para la encriptación simétrica. Ambas partes generan un par de claves temporal e intercambian la clave pública para poder generar el "shared secret" (clave simétrica) que se usará posteriormente.
* En la autenticación del cliente con el servidor.

### 5.2

Explicar los dos métodos de autenticación que existen.

#### Con contraseña

Es el método más simple. Funciona así:

1. El servidor pide al cliente que introduzca la contraseña del usuario con el que se están intentando conectar.
2. Esta contraseña se envía usando la encriptación previamente negociada, para que se mantenga en secreto frente a terceros.

#### Con par de claves

Es el método más popular y recomendado. Funciona así:

1. El cliente envía el ID del par de claves con el que se quiere autenticar ante el servidor.
2. El servidor comprueba el fichero `authorized_keys` del usuario al que el cliente está intentando conectarse, para comprobar si existe la clave correspondiente al ID que envió el cliente en el paso anterior.
3. Si se encuentra una clave pública que corresponda con el ID enviado, el servidor genera un número aleatorio y usa esa clave pública para encriptarlo.
4. El servidor envía al cliente este número encriptado.
5. Si el cliente tiene la clave privada asociada, podrá desencriptar este mensaje y revelar el número original.
6. El cliente combina el número desencriptado con la clave compartida de sesión que se está usando para encriptar la comunicación, y calcula el hash MD5 de este valor.
7. El cliente envía este hash de vuelta al servidor como respuesta al mensaje del número encriptado.
8. El servidor usa la misma clave compartida de sesión junto con el número original que envió al cliente, y calcula el hash MD5 él también.
9. El servidor compara el hash que él mismo generó con el que el cliente le envió. Si estos valores son iguales, se prueba que el cliente tiene la clave privada correcta y finalmente, este cliente es autenticado correctamente.

### 5.3

> Explicar la función del fichero `~/.ssh/known_hosts`

Es un fichero de cliente que contiene todas las claves públicas de los servidores a los que nos conectamos.

El cliente SSH usa este fichero para autenticar los servidores a los que se conecta.

### 5.4

> Explicar el significado del siguiente mensaje que aparece la primera vez que nos conectamos a un servidor

```console
$ ssh debian@172.22.200.74
The authenticity of host '172.22.200.74 (172.22.200.74)' can't be established.
ECDSA key fingerprint is SHA256:7ZoNZPCbQTnDso1meVSNoKszn38ZwUI4i6saebbfL4M.
Are you sure you want to continue connecting (yes/no)?
```

Nos indica que, al ser la primera vez que nos estamos conectando a este servidor, aún no tenemos su clave pública almacenada en el fichero `~/.ssh/known_hosts`. Por lo tanto, aún "no podemos" verificar la autenticidad del mismo.

Si estamos seguros de su autenticidad, diremos que sí, y eso almacenará su clave pública en `~/.ssh/known_hosts`.

### 5.5

> ¿Qué significa el siguiente mensaje que aparece cuando reutilizamos una ip?

```console
 $ ssh debian@172.22.200.74
 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
 @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
 IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
 Someone could be eavesdropping on you right now (man-in-the-middle attack)!
 It is also possible that a host key has just been changed.
 The fingerprint for the ECDSA key sent by the remote host is
 SHA256:W05RrybmcnJxD3fbwJOgSNNWATkVftsQl7EzfeKJgNc.
 Please contact your system administrator.
 Add correct host key in /home/jose/.ssh/known_hosts to get rid of this message.
 Offending ECDSA key in /home/jose/.ssh/known_hosts:103
   remove with:
   ssh-keygen -f "/home/jose/.ssh/known_hosts" -R "172.22.200.74"
 ECDSA host key for 172.22.200.74 has changed and you have requested strict checking.
```

Este mensaje simplemente indica que la clave pública del servidor al que nos estamos intentando conectar ha cambiado con respecto a la clave que teníamos guardada para esa IP en nuestro `known_hosts` previamente.

Se nos avisa porque es una forma de ataque man-in-the-middle, pero si estamos seguros de que somos nosotros mismos reutilizando una IP, simplemente borraremos la línea correspondiente en nuestro `known_hosts` sobre la antigua conexión y volveremos a conectarnos.

### 5.6

> ¿Qué guardamos y para qué sirve el fichero en el servidor `~/.ssh/authorized_keys`?

Este es un fichero de servidor, diferente por cada usuario del sistema, y guarda todas las claves públicas de clientes permitidos para conectarse.

Sirve para que el servidor verifique que los clientes conectándose sean legítimos (que tengan la clave privada correspondiente).
