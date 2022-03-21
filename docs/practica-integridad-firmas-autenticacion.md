# Práctica: Integridad, firmas y autenticación

## Tarea 1: Firmas electrónicas

### Ejercicio 1
> Enviar un fichero y la firma del mismo a Carlos

Enviaré:
```
-rw-r--r-- 1 vagrant vagrant   17 Nov 25 12:13 paracarlos.txt
```

Genero su firma:
```
gpg --detach-sign paracarlos.txt
```

Obtengo:
```
-rw-r--r-- 1 vagrant vagrant  438 Nov 25 12:14 paracarlos.txt.sig
```

> Verificar la firma recibida

Recibo:
```
-rw-r--r-- 1 vagrant vagrant    4 Nov 25 12:18 taninonino.txt
-rw-r--r-- 1 vagrant vagrant  438 Nov 25 12:18 taninonino.txt.sig
```

Verifico:
```
gpg --verify taninonino.txt.sig taninonino.txt
```
```
gpg: Signature made Thu Nov 25 12:12:20 2021 UTC
gpg:                using RSA key B446862585616A3FBC935F9A4DFDE0A4BA1C00D5
gpg: Good signature from "Carlos Rivero <carlosrivero1988@gmail.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: B446 8625 8561 6A3F BC93  5F9A 4DFD E0A4 BA1C 00D5
```

### Ejercicio 2
> ¿Qué significa el mensaje que aparece en el momento de verificar la firma?

```
 gpg: Firma correcta de "Pepe D <josedom24@gmail.com>" [desconocido]
 gpg: ATENCIÓN: ¡Esta clave no está certificada por una firma de confianza!
 gpg:          No hay indicios de que la firma pertenezca al propietario.
 Huellas dactilares de la clave primaria: E8DD 5DA9 3B88 F08A DA1D  26BF 5141 3DDB 0C99 55FC
```





### Ejercicio 3
> Crearemos un anillo de confianza entre los compañeros

#### Parte 1
> Subir clave pública a `pgp.rediris.es`

```
gpg --keyserver pgp.rediris.es --send-key F0B9E3109E86B53F6C0791E43A110CF1BEBCA39E
```

#### Parte 2
> Pasar el fingerprint a tus compañeros para que puedan descargarse tu clave pública

Hecho.

#### Parte 3
> Bajar tres claves públicas de compañeros. Firmar estas claves.

Me bajo la de Carlos:
```
gpg --keyserver pgp.rediris.es --recv-keys 4DFDE0A4BA1C00D5
```

Firmo la de Carlos:
```
gpg --sign-key B446862585616A3FBC935F9A4DFDE0A4BA1C00D5
```

Muestro mi firma en la clave de Carlos:

![](https://i.imgur.com/uAaPgyo.png)


Me bajo la de María Jesús:
```
gpg --keyserver pgp.rediris.es --recv-keys BE770D0718D1F998
```

Firmo la de María Jesús:
```
gpg --sign-key bpmariajesus20@gmail.com
```

Muestro mi firma en la clave de María Jesús:

![](https://i.imgur.com/rXHWWeX.png)


Me bajo la de Miguel:
```
gpg --keyserver pgp.rediris.es --recv-keys 93E00F9A8C74FBC0
```

Firmo la de Miguel:
```
gpg --sign-key miguelcor.rrss@gmail.com
```

Muestro mi firma en la clave de Miguel:

![](https://i.imgur.com/J1YYQ9r.png)

#### Parte 4
> Tu clave pública tiene que tener 3 firmas

En mi keybox:
```
gpg --list-sigs adristudy@gmail.com
```
```
pub   rsa3072 2021-11-25 [SC] [expires: 2023-11-25]
      F0B9E3109E86B53F6C0791E43A110CF1BEBCA39E
uid           [ultimate] Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sig 3        3A110CF1BEBCA39E 2021-11-25  Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sig          4DFDE0A4BA1C00D5 2021-11-25  Carlos Rivero <carlosrivero1988@gmail.com>
sig          0A3D7E065504F64B 2021-11-25  Daniel Parrales Garcia (Para ASIR) <daniparrales16@gmail.com>
sig          51D0DEC846173F6A 2021-11-25  Lara Pruna Ternero <larapruter@gmail.com>
sub   rsa3072 2021-11-25 [E] [expires: 2023-11-25]
sig          3A110CF1BEBCA39E 2021-11-25  Adrián Jaramillo Rodríguez <adristudy@gmail.com>
```

En `pgp.rediris.es`:

![](https://i.imgur.com/Y59vjg0.png)

#### Parte 5
> Al firmar claves, devolverlas a su dueño

Las exporto de esta manera:
```
gpg --armor --output carlos_rivero_firmado_adri.asc --export carlosrivero1988@gmail.com
```

#### Parte 6
> Subir a `pgp.rediris.es` la clave cuando tenga tres firmas

```
gpg --keyserver pgp.rediris.es --send-key F0B9E3109E86B53F6C0791E43A110CF1BEBCA39E
```

Muestro las 3 firmas en el keyserver:

![](https://i.imgur.com/rWPO617.png)

> Rellenar datos de tu clave pública [aquí](https://dit.gonzalonazareno.org/redmine/projects/asir2/wiki/Claves_p%C3%BAblicas_PGP_2021-2022)

![](https://i.imgur.com/v3FHprc.png)

#### Parte 7
> Bajar las claves públicas de tus compañeros con tres firmas

Daniel:
```
gpg --keyserver pgp.rediris.es --recv-key 0A3D7E065504F64B
```

Lara:
```
gpg --keyserver pgp.rediris.es --recv-key 51D0DEC846173F6A
```

Carlos:
```
gpg --keyserver pgp.rediris.es --recv-key 4DFDE0A4BA1C00D5
```

*(todas esas claves tienen 3 firmas originalmente en el servidor, pero al intentar descargarlas gpg borra las firmas)*

### Ejercicio 4
> Mostrar las firmas de tu clave pública

```
gpg --list-sigs adristudy@gmail.com
```
```
pub   rsa3072 2021-11-25 [SC] [expires: 2023-11-25]
      F0B9E3109E86B53F6C0791E43A110CF1BEBCA39E
uid           [ultimate] Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sig 3        3A110CF1BEBCA39E 2021-11-25  Adrián Jaramillo Rodríguez <adristudy@gmail.com>
sig          4DFDE0A4BA1C00D5 2021-11-25  Carlos Rivero <carlosrivero1988@gmail.com>
sig          0A3D7E065504F64B 2021-11-25  Daniel Parrales Garcia (Para ASIR) <daniparrales16@gmail.com>
sig          51D0DEC846173F6A 2021-11-25  Lara Pruna Ternero <larapruter@gmail.com>
sub   rsa3072 2021-11-25 [E] [expires: 2023-11-25]
sig          3A110CF1BEBCA39E 2021-11-25  Adrián Jaramillo Rodríguez <adristudy@gmail.com>
```

### Ejercicio 5
> Comprobar que se puede verificar "sin problemas” la firma de Carlos, porque ya se confía

```
gpg --verify taninonino.txt.sig taninonino.txt
```
```
gpg: Signature made Thu Nov 25 12:12:20 2021 UTC
gpg:                using RSA key B446862585616A3FBC935F9A4DFDE0A4BA1C00D5
gpg: Good signature from "Carlos Rivero <carlosrivero1988@gmail.com>" [full]
```

### Ejercicio 6
> Comprueba que puedes verificar con confianza una firma de una persona en las que no confías, pero sin embargo si confía otra persona en la que tu tienes confianza total.

mirar mi documentación antigua






## Tarea 2: Correo seguro con evolution

Ahora vamos a configurar nuestro cliente de correo electrónico para poder mandar correos cifrados, para ello:

### Parte 1
> Configurar el cliente de correo evolution con tu cuenta de correo habitual

```
sudo apt install evolution
```

![](https://i.imgur.com/S75Ga85.png)

En gnome previamente había añadido adristudy@gmail.com como mi cuenta de correo, así que Evolution se ha autoconfigurado con esa cuenta al instalarse.










### Parte 2
> Añade a la cuenta las opciones de seguridad para poder enviar correos firmados con tu clave privada


Indico mi clave privada para firmar:

![](https://i.imgur.com/EzbBEeH.png)


> Configurar opciones de seguridad para poder cifrar correos para otros destinatarios




### Parte 3
> Enviar un correo con tus compañeros y comprueba el funcionamiento adecuado de GPG

Envío un correo a Carlos:
![](https://i.imgur.com/Yxc5hAX.png)

Este es el correo enviado:
![](https://i.imgur.com/znoJDqT.png)

No lo puedo ver, porque está cifrado con la clave de Carlos, y yo no lo puedo descifrar porque no tengo su privada.

Este es el correo que le ha llegado a Carlos:
![](https://i.imgur.com/5Yq8DNm.png)

> Recibir un correo de Carlos y comprobar el funcionamiento adecuado de GPG

Este es el correo que he recibido:

![](https://i.imgur.com/sEaWbLf.png)

Se ha desencriptado correctamente, y la firma se ha verificado.



### Parte X
> Mandar a Raúl un correo cifrado y firmado

Me bajo la clave de Raúl a mi keybox:
```
gpg --keyserver pgp.rediris.es --recv-key A036DF623D608DE0
```
```
gpg: key A036DF623D608DE0: public key "Raúl Ruiz Padilla <rruizp@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Si dejo marcada esa opción, no será necesario firmar las claves públicas de las personas a quienes queramos enviar correos:
![](https://i.imgur.com/AdGK5p7.png)

Prueba de que he enviado el correo:

![](https://i.imgur.com/oq8629Y.png)

Me aparece ese error porque está encriptado con la clave pública de Raúl, y como no tengo la privada asociada no prueba desencriptarlo.

Eso significa que se ha encriptado correctamente.






## Tarea 3: Integridad de ficheros

### Parte 1

> Descargar la ISO de Debian

```
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-11.2.0-amd64-netinst.iso
```

Obtengo el fichero:
```
-rw-r--r-- 1 vagrant vagrant 396361728 Dec 18 13:24 debian-11.2.0-amd64-netinst.iso
```



### Parte 2
> Comprobar la integridad de la ISO descargada usando `sha512sum`

En el fichero [SHA512SUMS](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS) que nos proporciona Debian, nos encontramos el siguiente checksum para la ISO descargada:
```
c685b85cf9f248633ba3cd2b9f9e781fa03225587e0c332aef2063f6877a1f0622f56d44cf0690087b0ca36883147ecb5593e3da6f965968402cdbdf12f6dd74
```

Luego utilizo `sha512sum` sobre la ISO que me he descargado:
```
sha512sum debian-11.2.0-amd64-netinst.iso
```
```
c685b85cf9f248633ba3cd2b9f9e781fa03225587e0c332aef2063f6877a1f0622f56d44cf0690087b0ca36883147ecb5593e3da6f965968402cdbdf12f6dd74  debian-11.2.0-amd64-netinst.iso
```

Ambos hashes coinciden, así que nos hemos descargado el fichero correctamente.

Igualmente, si queremos evitar errores, podemos usar [una herramienta online](https://text-compare.com/) para comparar strings:

![](https://i.imgur.com/KUhapSW.png)



### Parte 3
> Verificar que `SHA512SUMS` no ha sido manipulado usando `SHA512SUMS.sign`

Primero descargo ambos ficheros:
```
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS.sign
```

Muestro que los tengo:
```
-rw-r--r-- 1 vagrant vagrant       494 Dec 18 20:42 SHA512SUMS
-rw-r--r-- 1 vagrant vagrant       833 Dec 18 20:45 SHA512SUMS.sign
```

Averiguo el fingerprint del par de claves que se usó originalmente para crear la firma `SHA512SUMS.sign`:
```
gpg --verify SHA512SUMS.sign
```
![](https://i.imgur.com/pSmi6pt.png)

No se puede verificar la firma porque no tenemos la clave pública de Debian en nuestro keybox, pero ya sabemos su fingerprint para descargarla:
```
gpg --keyserver keyring.debian.org --recv DF9B9C49EAA9298432589D76DA87E80D6294BE9B
```
```
gpg: key DA87E80D6294BE9B: public key "Debian CD signing key <debian-cd@lists.debian.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Finalmente, podemos verificar la firma:
```
gpg --verify SHA512SUMS.sign SHA512SUMS
```
```
gpg: Signature made Sat Dec 18 20:45:36 2021 UTC
gpg:                using RSA key DF9B9C49EAA9298432589D76DA87E80D6294BE9B
gpg: Good signature from "Debian CD signing key <debian-cd@lists.debian.org>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: DF9B 9C49 EAA9 2984 3258  9D76 DA87 E80D 6294 BE9B
```

Se ha podido verificar correctamente, `SHA512SUMS` es legítimo.






## Tarea 4: Integridad y autenticidad (apt-secure)

### Parte 1
> ¿Qué herramienta utiliza apt-secure para firmar y verificar firmas?

GPG.



### Parte 2
> ¿Para qué sirve `apt-key`?

Se usa para gestionar la lista de claves que mantiene apt para autenticar paquetes.  

Los paquetes que sean autenticados usando estas claves se considerarán "trusted".

> ¿Qué muestra el comando `apt-key list`?

Lista las claves públicas en las que confiamos.  

Muestro mi output de `apt-key list`:
```
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



### Parte 3
> ¿Dónde se encuentran los distintos keyrings que usa `apt-key`?

En `/etc/apt/trusted.gpg.d/`.



### Parte 4
> ¿Qué contiene el fichero `Release` de un repositorio?

En otros datos, contiene los checksums de los ficheros que hay en el repositorio.

Ejemplo: http://ftp.debian.org/debian/dists/Debian11.2/Release

> ¿Y `Release.gpg`?

Contiene la firma detached ascii-armored del fichero `Release`.

Ejemplo: http://ftp.debian.org/debian/dists/Debian11.2/Release.gpg



### Parte 5
> Explicar el proceso por el cual apt nos asegura que los ficheros que estamos descargando son legítimos

1. Se descargan los ficheros `Release`, `Release.gpg`, y el fichero `Packages` que corresponda.
2. Se comprueba usando la firma `Release.gpg` más la clave pública correspondiente, que el fichero `Release` es legítimo. A partir de aquí, apt ya confía en todos sus checksums.
3. Apt compara el checksum del fichero `Packages` con el que encuentra en el fichero `Release`; si coinciden, quiere decir que se descargó el fichero correcto.
4. Al descargarnos un paquete individual se compara su checksum con el que aparece en `Packages`; si coinciden, quiere decir que se descargó el paquete correcto.



### Parte 6
> Añadir el repositorio de VirtualBox

Añado las claves gpg necesarias:
```
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
```

Añado el repositorio:
```
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




























### Parte 4
> ¿Qué significa este mensaje que aparece la primera vez que nos conectamos a un servidor?
```
$ ssh debian@172.22.200.74
The authenticity of host '172.22.200.74 (172.22.200.74)' can't be established.
ECDSA key fingerprint is SHA256:7ZoNZPCbQTnDso1meVSNoKszn38ZwUI4i6saebbfL4M.
Are you sure you want to continue connecting (yes/no)?
```



### Parte 5
> En ocasiones cuando estamos trabajando en el cloud, y reutilizamos una ip flotante nos aparece este mensaje:

```
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


### Parte 6
> ¿Qué guardamos y para qué sirve el fichero en el servidor ~/.ssh/authorized_keys?

























