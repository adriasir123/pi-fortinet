# Práctica: Cifrado asimétrico con gpg y openssl

## Tarea 1: Generación de claves

### Ejercicio 0
> Instalar gpg

```
sudo apt update
sudo apt install gnupg
```

### Ejercicio 1
> Generar par de claves

```
gpg --gen-key
```

![](https://i.imgur.com/hQJG0dl.png)

En rojo, marco lo que yo he tenido que escribir.

Cuando pide una passphrase, no le pongo ninguna.

> ¿En qué directorio se guardan las claves de un usuario?

`/home/vagrant/.gnupg`

### Ejercicio 2
> Listar las claves públicas

```
gpg --list-keys
```
```
/home/vagrant/.gnupg/pubring.kbx
--------------------------------
pub   rsa3072 2021-11-15 [SC] [expires: 2023-11-15]
      5FAB740108563CA70D25EFA770B15EFFA5837AF8
uid           [ultimate] Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sub   rsa3072 2021-11-15 [E] [expires: 2023-11-15]
```

> Explicar el output

- `/home/vagrant/.gnupg/pubring.kbx`: anillo de claves
- `pub`: indicador de clave pública
- `rsa3072`: tipo de clave rsa y tamaño en bits de 3072
- `2021-11-15`: fecha de creación
- `[expires: 2023-11-15]`: fecha de expiración
- `5FAB740108563CA70D25EFA770B15EFFA5837AF8`: fingerprint de la clave
- `uid`: identificadores de la clave
- `sub`: indica una "subkey". Esto es un concepto un poco avanzado de gpg que no hemos visto, porque en realidad cada par de claves que creamos a su vez crean subpares de claves por debajo.

> ¿Cómo deberías haber generado las claves para indicar, por ejemplo, que tengan 1 mes de validez?

```
gpg --full-gen-key
```

![](https://i.imgur.com/62UcDKR.png)

En rojo he marcado cuando:

- Defino 1 mes de validez
- Aparece la fecha de expiración correcta en la clave

### Ejercicio 3
> Listar las claves privadas

```
gpg --list-secret-keys
```
```
/home/vagrant/.gnupg/pubring.kbx
--------------------------------
sec   rsa3072 2021-11-15 [SC] [expires: 2023-11-15]
      5FAB740108563CA70D25EFA770B15EFFA5837AF8
uid           [ultimate] Adrián Jaramillo Rodríguez <adristudy@gmail.com>
ssb   rsa3072 2021-11-15 [E] [expires: 2023-11-15]
```





## Tarea 2: Importar / exportar clave pública

### Ejercicio 1
> Exportar tu clave pública en ASCII al fichero `adrian_jaramillo.asc`

```
gpg --armor --output adrian_jaramillo.asc --export adristudy@gmail.com
```

> Enviar `adrian_jaramillo.asc` a Carlos

Enviado.

### Ejercicio 2
> Importar la clave pública recibida de Carlos

He recibido:
```
-rw-r--r-- 1 vagrant vagrant 2476 Nov 15 09:18 carlos_rivero.asc
```

La importo:
```
gpg --import carlos_rivero.asc
```
```
gpg: key 69700115DE666C54: public key "Carlos Rivero Martín <carlosrivero1988@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

### Ejercicio 3
> Comprobar que la clave se ha incluido correctamente

```
gpg --list-keys
```
```
/home/vagrant/.gnupg/pubring.kbx
--------------------------------
pub   rsa3072 2021-11-15 [SC] [expires: 2023-11-15]
      5FAB740108563CA70D25EFA770B15EFFA5837AF8
uid           [ultimate] Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sub   rsa3072 2021-11-15 [E] [expires: 2023-11-15]

pub   rsa3072 2021-11-15 [SC] [expires: 2023-11-15]
      35AAEDB27BDF2918D11FB3B569700115DE666C54
uid           [ unknown] Carlos Rivero Martín <carlosrivero1988@gmail.com>
sub   rsa3072 2021-11-15 [E] [expires: 2023-11-15]
```

Abajo tenemos la de Carlos.





## Tarea 3: Cifrado asimétrico con claves públicas

### Ejercicio 1
> Cifrar un fichero con la clave pública de Carlos

```
gpg --output para_carlos.txt.gpg --encrypt --recipient carlosrivero1988@gmail.com para_carlos.txt
```
```
gpg: A226D42A5B0599A6: There is no assurance this key belongs to the named user

sub  rsa3072/A226D42A5B0599A6 2021-11-15 Carlos Rivero Martín <carlosrivero1988@gmail.com>
 Primary key fingerprint: 35AA EDB2 7BDF 2918 D11F  B3B5 6970 0115 DE66 6C54
      Subkey fingerprint: 0FC1 7DA5 8FC3 9973 A4FA  5648 A226 D42A 5B05 99A6

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N) y
```

Fichero generado:
```
-rw-r--r-- 1 vagrant vagrant  498 Nov 15 09:46 para_carlos.txt.gpg
```

> Enviar el fichero cifrado a Carlos

Enviado.

### Ejercicio 2
> Carlos nos enviará un fichero cifrado

```
-rw-r--r-- 1 vagrant vagrant  502 Nov 15 09:58 para_adrian.txt.gpg
```

### Ejercicio 3
> Descifrar `para_adrian.txt.gpg`

```
gpg --output para_adrian.txt --decrypt para_adrian.txt.gpg
```
```
gpg: encrypted with 3072-bit RSA key, ID ABA647DE60867CB5, created 2021-11-15
      "Adrián Jaramillo Rodríguez <adristudy@gmail.com>"
```
Se nos indica la clave pública original con la que se encriptó el mensaje.

Fichero desencriptado:
```
-rw-r--r-- 1 vagrant vagrant   30 Nov 15 10:11 para_adrian.txt
```

Contenido:
```
vagrant@practicacifradogpg:~$ cat para_adrian.txt
Mensaje cifrado para Adrián.
```

### Ejercicio 4
> Enviar `para_carlos.txt.gpg` a Daniel Parrales

Enviado.

> Comprobar que Daniel no puede descifrar `para_carlos.txt.gpg`

Si intenta hacer:
```
gpg --output para_carlos.txt --decrypt para_carlos.txt.gpg
```

Le aparece:
```
gpg: encrypted with RSA key, ID A226D42A5B0599A6
gpg: decryption failed: No secret key
```

Esto sucede porque Daniel no tiene la clave privada asociada a la pública con la que originalmente se encriptó el fichero.

### Ejercicio 5
> Borrar tu clave privada

```
gpg --delete-secret-key adristudy@gmail.com
```

Muestro que se ha borrado:
```
vagrant@practicacifradogpg:~$ gpg --list-secret-keys
vagrant@practicacifradogpg:~$
```

> Borrar todas las claves públicas

```
gpg --delete-key carlosrivero1988@gmail.com
gpg --delete-key adristudy@gmail.com
```

Muestro que no queda ninguna:
```
vagrant@practicacifradogpg:~$ gpg --list-keys
vagrant@practicacifradogpg:~$
```


## Tarea 4: Exportar clave a un servidor público de claves PGP

### Ejercicio 1
> Generar certificado de revocación de tu clave pública

```
gpg --output revoke_adrianj.asc --gen-revoke adristudy@gmail.com
```

Seguimos los pasos que se nos indican *(en rojo, mis respuestas)*:

![](https://i.imgur.com/duHO4i3.png)

Fichero generado:
```
-rw------- 1 vagrant vagrant  719 Nov 15 19:25 revoke_adrianj.asc
```

### Ejercicio 2
> Exportar tu clave pública a `pgp.rediris.es`

```
gpg --keyserver pgp.rediris.es --send-key 5FAB740108563CA70D25EFA770B15EFFA5837AF8
```
```
gpg: sending key 70B15EFFA5837AF8 to hkp://pgp.rediris.es
```

**Obligatorio** indicar el "key ID" después de `--send-key` *(no funcionaría, por ejemplo, indicando un correo).*

Compruebo que se haya subido correctamente:
```
gpg --keyserver pgp.rediris.es --search-key adristudy@gmail.com
```

![](https://i.imgur.com/T4HswxU.png)

La número 1 es la que acabo de subir *(las demás claves son de otros años)*.

La sé identificar por:

- ID resumido (últimos 16 caracteres del ID original)
- Fecha de creación

### Ejercicio 3
> Borrar la clave pública de Carlos

```
gpg --delete-keys carlosrivero1988@gmail.com
```
```
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  rsa3072/69700115DE666C54 2021-11-15 Carlos Rivero Martín <carlosrivero1988@gmail.com>

Delete this key from the keyring? (y/N) y
```

> Importarla desde `pgp.rediris.es`

Para importar su clave pública de nuevo a mi keybox, primero, tengo que averiguar su ID.

```
gpg --keyserver pgp.rediris.es --search-key carlosrivero1988@gmail.com
```
```
gpg: data source: http://130.206.1.8:11371
(1)	Carlos Rivero Martín <carlosrivero1988@gmail.com>
	  3072 bit RSA key 69700115DE666C54, created: 2021-11-15, expires: 2023-11-15
```

Sabiendo su ID, `69700115DE666C54`, me bajo la clave:
```
gpg --keyserver pgp.rediris.es --recv-key 69700115DE666C54
```
```
gpg: key 69700115DE666C54: public key "Carlos Rivero Martín <carlosrivero1988@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Vuelvo a listar mis claves públicas, y la tengo:
```
vagrant@practicacifradogpg:~$ gpg --list-key
/home/vagrant/.gnupg/pubring.kbx
--------------------------------
pub   rsa3072 2021-11-15 [SC] [expires: 2023-11-15]
      5FAB740108563CA70D25EFA770B15EFFA5837AF8
uid           [ultimate] Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sub   rsa3072 2021-11-15 [E] [expires: 2023-11-15]

pub   rsa3072 2021-11-15 [SC] [expires: 2023-11-15]
      35AAEDB27BDF2918D11FB3B569700115DE666C54
uid           [ unknown] Carlos Rivero Martín <carlosrivero1988@gmail.com>
sub   rsa3072 2021-11-15 [E] [expires: 2023-11-15]
```





## Tarea 5: Cifrado asimétrico con openssl

### Ejercicio 1
> Generar par de claves

```
openssl genrsa -des3 -out keypair.pem 2048
```
```
Generating RSA private key, 2048 bit long modulus (2 primes)
......................................................................................+++++
....................................................................................................................+++++
e is 65537 (0x010001)
Enter pass phrase for keypair.pem:
Verifying - Enter pass phrase for keypair.pem:
```

Passphrase: 1234

Fichero resultante:
```
-rw------- 1 vagrant vagrant 1743 Nov 15 22:56 keypair.pem
```

### Ejercicio 2
> Enviar tu clave pública a Carlos

Exporto mi clave pública del fichero `keypair.pem`:
```
openssl rsa -in keypair.pem -outform PEM -pubout -out public_adrianj.pem
```
```
Enter pass phrase for keypair.pem:
writing RSA key
```

Nos pregunta la passphrase, y nos genera:
```
-rw-r--r-- 1 vagrant vagrant  451 Nov 15 23:15 public_adrianj.pem
```

Este fichero, es el que envío a Carlos.

### Ejercicio 3
> Recibir la clave pública de Carlos

```
-rw-r--r-- 1 vagrant vagrant  451 Nov 16 00:08 public_crivero.pem
```

> Utilizando esa clave, cifra un fichero de texto y envíaselo

```
openssl rsautl -encrypt -inkey public_crivero.pem -pubin -in para_carlos_openssl.txt -out para_carlos_openssl.txt.enc
```

Nos genera este fichero cifrado:
```
-rw-r--r-- 1 vagrant vagrant  256 Nov 16 00:24 para_carlos_openssl.txt.enc
```

Se lo envío.

### Ejercicio 4
> Recibe un fichero cifrado de Carlos

```
-rw-r--r-- 1 vagrant vagrant  256 Nov 16 00:35 para_adrian_openssl.txt.enc
```

> Descifra el fichero

```
openssl rsautl -decrypt -inkey keypair.pem -in para_adrian_openssl.txt.enc > para_adrian_openssl.txt
```
```
Enter pass phrase for keypair.pem:
```

Introducimos la passphrase, y obtenemos el fichero:
```
-rw-r--r-- 1 vagrant vagrant   58 Nov 16 00:43 para_adrian_openssl.txt
```

Con el contenido:
```
vagrant@practicacifradogpg:~$ cat para_adrian_openssl.txt
Este mensaje se cifrará con openssl.

Es para Adrián.
```
