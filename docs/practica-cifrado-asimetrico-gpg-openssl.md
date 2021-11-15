# Práctica: Cifrado asimétrico con gpg y openssl

## Tarea 1: Generación de claves

Los algoritmos de cifrado asimétrico utilizan dos claves para el cifrado y descifrado de mensajes. Cada persona involucrada (receptor y emisor) debe disponer, por tanto, de una pareja de claves pública y privada. Para generar nuestra pareja de claves con gpg utilizamos la opción --gen-key:

Para esta práctica no es necesario que indiquemos frase de paso en la generación de las claves (al menos para la clave pública).


### Ejercicio 1
> Genera un par de claves (pública y privada). ¿En que directorio se guarda las claves de un usuario?

### Ejercicio 2
> Lista las claves públicas que tienes en tu almacén de claves. Explica los distintos datos que nos muestra. ¿Cómo deberías haber generado las claves para indicar, por ejemplo, que tenga un 1 mes de validez?

### Ejercicio 3
> Lista las claves privadas de tu almacén de claves.




## Tarea 2: Importar / exportar clave pública

Para enviar archivos cifrados a otras personas, necesitamos disponer de sus claves públicas. De la misma manera, si queremos que cierta persona pueda enviarnos datos cifrados, ésta necesita conocer nuestra clave pública. Para ello, podemos hacérsela llegar por email por ejemplo. Cuando recibamos una clave pública de otra persona, ésta deberemos incluirla en nuestro keyring o anillo de claves, que es el lugar donde se almacenan todas las claves públicas de las que disponemos.

### Ejercicio 1
> Exporta tu clave pública en formato ASCII y guardalo en un archivo nombre_apellido.asc y envíalo al compañero con el que vas a hacer esta práctica.

### Ejercicio 2
> Importa las claves públicas recibidas de vuestro compañero.

### Ejercicio 3
> Comprueba que las claves se han incluido correctamente en vuestro keyring.




## Tarea 3: Cifrado asimétrico con claves públicas

Tras realizar el ejercicio anterior, podemos enviar ya documentos cifrados utilizando la clave pública de los destinatarios del mensaje.

### Ejercicio 1
> Cifraremos un archivo cualquiera y lo remitiremos por email a uno de nuestros compañeros que nos proporcionó su clave pública.

### Ejercicio 2
> Nuestro compañero, a su vez, nos remitirá un archivo cifrado para que nosotros lo descifremos.

### Ejercicio 3
> Tanto nosotros como nuestro compañero comprobaremos que hemos podido descifrar los mensajes recibidos respectivamente.

### Ejercicio 4
> Por último, enviaremos el documento cifrado a alguien que no estaba en la lista de destinatarios y comprobaremos que este usuario no podrá descifrar este archivo.

### Ejercicio 5
> Para terminar, indica los comandos necesarios para borrar las claves públicas y privadas que posees.




## Tarea 4: Exportar clave a un servidor público de claves PGP

Para distribuir las claves públicas es mucho más habitual utilizar un servidor específico para distribuirlas, que permite a los clientes añadir las claves públicas a sus anillos de forma mucho más sencilla.



### Ejercicio 1
> Genera la clave de revocación de tu clave pública para utilizarla en caso de que haya problemas.

### Ejercicio 2
> Exporta tu clave pública al servidor pgp.rediris.es

### Ejercicio 3
> Borra la clave pública de alguno de tus compañeros de clase e impórtala ahora del servidor público de rediris.





## Tarea 5: Cifrado asimétrico con openssl

En esta ocasión vamos a cifrar nuestros ficheros de forma asimétrica utilizando la herramienta openssl.

### Ejercicio 1
> Genera un par de claves (pública y privada).

### Ejercicio 2
> Envía tu clave pública a un compañero.

### Ejercicio 3
> Utilizando la clave pública cifra un fichero de texto y envíalo a tu compañero.

### Ejercicio 4
> Tu compañero te ha mandado un fichero cifrado, muestra el proceso para el descifrado.
