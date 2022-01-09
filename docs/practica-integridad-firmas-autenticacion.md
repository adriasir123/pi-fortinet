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













### Parte 3

Verifica que el contenido del hash que has utilizado no ha sido manipulado, usando la firma digital que encontrarás en el repositorio. Puedes encontrar una guía para realizarlo en este artículo: How to verify an authenticity of downloaded Debian ISO images























end
