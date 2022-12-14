---
title: "Práctica: Cifrado asimétrico con gpg y openssl"
---

# **Tarea 1**: Generación de claves 

## 1. Genera un par de claves (pública y privada). ¿En que directorio se guardan las claves de un usuario?

```
gpg --gen-key
```
```
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: directory '/home/vagrant/.gnupg' created
gpg: keybox '/home/vagrant/.gnupg/pubring.kbx' created
Note: Use "gpg --full-generate-key" for a full featured key generation dialog.

GnuPG needs to construct a user ID to identify your key.

Real name: Adrián Jaramillo Rodríguez
Email address: adristudy@gmail.com
You are using the 'utf-8' character set.
You selected this USER-ID:
    "Adrián Jaramillo Rodríguez <adristudy@gmail.com>"

Change (N)ame, (E)mail, or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: /home/vagrant/.gnupg/trustdb.gpg: trustdb created
gpg: key 7E0C48CBBC17F36F marked as ultimately trusted
gpg: directory '/home/vagrant/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/vagrant/.gnupg/openpgp-revocs.d/C34C081A07CA71291F82BFD67E0C48CBBC17F36F.rev'
public and secret key created and signed.

pub   rsa3072 2020-10-08 [SC] [expires: 2022-10-08]
      C34C081A07CA71291F82BFD67E0C48CBBC17F36F
uid                      Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sub   rsa3072 2020-10-08 [E] [expires: 2022-10-08]
```

decir en qué directorio se guardan las claves



























# **Tarea 2**: Importar / exportar clave pública 
 
## 1. Exporta tu clave pública en formato ASCII y guardalo en un archivo nombre_apellido.asc y envíalo al compañero con el que vas a hacer esta práctica.

```
gpg --output adrian_jaramillo.asc --armor --export adristudy@gmail.com
```


## 2. Importa las claves públicas recibidas de vuestro compañero

```
gpg --import jose_calderon.asc
```

```
gpg: key A52A681834F0E596: public key "José Miguel Calderón Frutos <josemiguelcalderonfrutos@gamil.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```



## 3. Comprueba que las claves se han incluido correctamente en vuestro keyring.

```
gpg --list-public-keys 
```

```
/home/vagrant/.gnupg/pubring.kbx
--------------------------------
pub   rsa3072 2020-10-08 [SC] [expires: 2022-10-08]
      C34C081A07CA71291F82BFD67E0C48CBBC17F36F
uid           [ultimate] Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sub   rsa3072 2020-10-08 [E] [expires: 2022-10-08]

pub   rsa3072 2020-10-07 [SC] [expires: 2020-11-07]
      DCFB091C5495684E59BC061EA52A681834F0E596
uid           [ unknown] José Miguel Calderón Frutos <josemiguelcalderonfrutos@gamil.com>
sub   rsa3072 2020-10-07 [E] [expires: 2022-10-07]
```

























# **Tarea 3**: Cifrado asimétrico con claves públicas 

## 1. Cifraremos un archivo cualquiera y lo remitiremos por email a uno de nuestros compañeros que nos proporcionó su clave pública.

```
gpg -e -r josemiguelcalderonfrutos@gamil.com para_jose_asimetrico.txt
```

Genera
```
-rw-r--r-- 1 vagrant vagrant  509 Oct 18 18:16 para_jose_asimetrico.txt.gpg
```

## 2. Nuestro compañero, a su vez, nos remitirá un archivo cifrado para que nosotros lo descifremos.

El fichero que me ha pasado Jose es...
```
-rw-r--r-- 1 vagrant vagrant  477 Oct 18 18:19 Pruebas.gpg
```

## 3. Tanto nosotros como nuestro compañero comprobaremos que hemos podido descifrar los mensajes recibidos respectivamente.

```
gpg --decrypt Pruebas.gpg > Pruebas.txt
```

Nos la clave pública con la que se cifró el fichero. Es la nuestra, claramente, para que sólo con nuestra clave privada este mensaje pudiera ser descifrado
```
gpg: encrypted with 3072-bit RSA key, ID 6C17AF823AD42CC6, created 2020-10-08
      "Adrián Jaramillo Rodríguez <adristudy@gmail.com>"
```

El contenido del fichero es...
```
vagrant@practica-gpg-openssl-asimetrico:~$ cat Pruebas.txt 
Hola adri
```































# **Tarea 4**: Exportar clave a un servidor público de claves PGP

## 1. Genera la clave de revocación de tu clave pública para utilizarla en caso de que haya problemas.

## 2. Exporta tu clave pública al servidor `pgp.rediris.es`

```
gpg --keyserver pgp.rediris.es --send-keys C34C081A07CA71291F82BFD67E0C48CBBC17F36F
```
```
gpg: sending key 7E0C48CBBC17F36F to hkp://pgp.rediris.es
```

## 3. Borra la clave pública de alguno de tus compañeros de clase e impórtala ahora del servidor público de rediris.

```
gpg --delete-key josemiguelcalderonfrutos@gamil.com
```

```
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  rsa3072/A52A681834F0E596 2020-10-07 José Miguel Calderón Frutos <josemiguelcalderonfrutos@gamil.com>

Delete this key from the keyring? (y/N) y
```

Importamos la clave
```
gpg --keyserver pgp.rediris.es --recv-keys DCFB091C5495684E59BC061EA52A681834F0E596
```
```
gpg: key A52A681834F0E596: public key "José Miguel Calderón Frutos <josemiguelcalderonfrutos@gamil.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

# **Tarea 5**: Cifrado asimétrico con openssl 

## 1. Genera un par de claves (pública y privada)

```
openssl genrsa -des3 -out keypair.pem 2048
```
> Contraseña: hola

Aquí lo tenemos generado
```
-rw------- 1 vagrant vagrant 1751 Oct 18 18:45 keypair.pem
```


## 2. Envía tu clave pública a un compañero

Exportamos del fichero anterior, nuestra clave pública
```
openssl rsa -in keypair.pem -outform PEM -pubout -out public_adri.pem
```

```
-rw-r--r-- 1 vagrant vagrant  451 Oct 18 18:52 public_adri.pem
```

Esa clave, la envío a Jose


## 3. Utilizando la clave pública cifra un fichero de texto y envíalo a tu compañero

```
openssl rsautl -encrypt -in para_jose_asimetrico_openssl.txt -out para_jose_asimetrico_openssl.txt.enc -inkey public_jose.pem -pubin
```

Fichero generado...
```
-rw-r--r-- 1 vagrant vagrant  256 Oct 18 18:59 para_jose_asimetrico_openssl.txt.enc

```

Se lo envío

## 4. Tu compañero te ha mandado un fichero cifrado, muestra el proceso para el descifrado

```
openssl rsautl -decrypt -inkey keypair.pem -in Pruba.enc -out Pruba.txt
```

```
vagrant@practica-gpg-openssl-asimetrico:~$ cat Pruba.txt 
Tonto el que lo lea
```









