# Práctica: Integridad, firmas y autenticación

## Tarea 1: Firmas electrónicas

### Ejercicio 1
> Manda un documento y la firma electrónica del mismo a un compañero.

```
gpg --detach-sign paracarlos.txt
```
```
-rw-r--r-- 1 vagrant vagrant   17 Nov 25 12:13 paracarlos.txt
-rw-r--r-- 1 vagrant vagrant  438 Nov 25 12:14 paracarlos.txt.sig
```

> Verifica la firma que tu has recibido.

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

 gpg: Firma correcta de "Pepe D <josedom24@gmail.com>" [desconocido]
 gpg: ATENCIÓN: ¡Esta clave no está certificada por una firma de confianza!
 gpg:          No hay indicios de que la firma pertenezca al propietario.
 Huellas dactilares de la clave primaria: E8DD 5DA9 3B88 F08A DA1D  26BF 5141 3DDB 0C99 55FC

### Ejercicio 3
> Vamos a crear un anillo de confianza entre los miembros de nuestra clase, para ello.






### Parte 1
> Tu clave pública debe estar en un servidor de claves

```
gpg --keyserver pgp.rediris.es --send-key F0B9E3109E86B53F6C0791E43A110CF1BEBCA39E
```

### Parte 2
> Escribe tu fingerprint en un papel y dárselo a tu compañero, para que puede descargarse tu clave pública.

Hecho.

### Parte 3
> Te debes bajar al menos tres claves públicas de compañeros. Firma estas claves.

```
gpg --sign-key B446862585616A3FBC935F9A4DFDE0A4BA1C00D5
```
```
pub  rsa3072/4DFDE0A4BA1C00D5
     created: 2021-11-25  expires: 2023-11-25  usage: SC  
     trust: unknown       validity: unknown
sub  rsa3072/54A887151550A718
     created: 2021-11-25  expires: 2023-11-25  usage: E   
[ unknown] (1). Carlos Rivero <carlosrivero1988@gmail.com>


pub  rsa3072/4DFDE0A4BA1C00D5
     created: 2021-11-25  expires: 2023-11-25  usage: SC  
     trust: unknown       validity: unknown
 Primary key fingerprint: B446 8625 8561 6A3F BC93  5F9A 4DFD E0A4 BA1C 00D5

     Carlos Rivero <carlosrivero1988@gmail.com>

This key is due to expire on 2023-11-25.
Are you sure that you want to sign this key with your
key "Adrián Jaramillo Rodríguez <adristudy@gmail.com>" (3A110CF1BEBCA39E)

Really sign? (y/N) y
```


### Parte 4
> Tu te debes asegurar que tu clave pública es firmada por al menos tres compañeros de la clase.

Recibo:
```
-rw-r--r-- 1 vagrant vagrant 3069 Nov 25 12:43 clave_adrian_firmada.asc
```

Importo:
```
gpg --import clave_adrian_firmada.asc
```



### Parte 5
> Una vez que firmes una clave se la tendrás que devolver a su dueño, para que otra persona se la firme.

```
gpg --armor --output carlos_rivero_firmado_adri.asc --export carlosrivero1988@gmail.com
```

Se la he devuelto.

### Parte 6
> Cuando tengas las tres firmas sube la clave al servidor de claves y rellena tus datos en la tabla Claves públicas PGP 2020-2021

Para mi keybox:
```
gpg --import clavefirmada_adri.asc
```

Para el keyserver:
```
gpg --keyserver pgp.rediris.es --send-key F0B9E3109E86B53F6C0791E43A110CF1BEBCA39E
```

```
gpg --list-sigs adristudy@gmail.com
```
```

```



### Parte 7
> Asegurate que te vuelves a bajar las claves públicas de tus compañeros que tengan las tres firmas.

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








### Ejercicio 4
> Muestra las firmas que tiene tu clave pública.

### Ejercicio 5
> Comprueba que ya puedes verificar sin “problemas” una firma recibida por una persona en la que confías.

### Ejercicio 6
> Comprueba que puedes verificar con confianza una firma de una persona en las que no confías, pero sin embargo si confía otra persona en la que tu tienes confianza total.


























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
