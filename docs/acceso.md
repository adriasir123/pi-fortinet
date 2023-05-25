# Acceso

Teniendo ya nuestro FortiGate de fábrica, ahora tendremos que acceder.

Si recordamos, un FortiGate podía ser instalado tanto de manera local como cloud, y en el propio dispositivo teníamos las instrucciones de acceso en una pegatina.

En nuestro caso todo el proyecto será local, así que estas serán las instrucciones pertinentes:

![12](images/primeros-pasos/12.jpeg)

## Paso 1

Conectamos el puerto 1 a nuestra máquina:

![13](images/primeros-pasos/13.jpeg)

El propio FortiGate tiene un DHCP integrado, podemos comprobarlo viendo la IP que nos ha dado:

![14](images/primeros-pasos/14.png)

## Paso 2

La IP a la que tenemos que acceder es 192.168.1.99, y podemos hacer una primera prueba de conectividad:

![15](images/primeros-pasos/15.png)

Viendo que funciona, ya podemos acceder desde el navegador:

![16](images/primeros-pasos/16.png)

## Paso 3

Las primeras credenciales son las siguientes:

![17](images/primeros-pasos/17.png)

*(Sí, sin contraseña)*

Tras entrar nos obliga a cambiar la contraseña:

![18](images/primeros-pasos/18.png)

Por ejemplo le podemos poner admin también de contraseña.

## Paso 4

Nos devuelve a la pantalla de login inicial, y tras entrar con las credenciales admin admin, nos pide una pequeña configuración inicial:

![19](images/primeros-pasos/19.png)

El hostname es adecuado:

![20](images/primeros-pasos/20.png)

Con el dashboard básico nos bastaría:

![21](images/primeros-pasos/21.png)

Ya tendríamos acceso por completo al FortiGate:

![22](images/primeros-pasos/22.png)

*(Falla la carga del vídeo de introducción porque aún no tenemos configurada la salida a Internet)*
