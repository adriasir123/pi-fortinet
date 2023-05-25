# Acceso

Teniendo ya nuestro FortiGate de fábrica, ahora tendremos que acceder.

Si recordamos, un FortiGate podía ser instalado tanto de manera local como cloud, y en el propio dispositivo teníamos las instrucciones de acceso en una pegatina.

En nuestro caso todo el proyecto será local, así que estas serán las instrucciones pertinentes:

![instrucciones-locales](images/instrucciones-locales.jpeg)

## Paso 1

Conectamos el puerto 1 a nuestra máquina:

![puerto1-a-portatil](images/puerto1-a-portatil.jpeg)

El propio FortiGate tiene un DHCP integrado, podemos comprobarlo viendo la IP que nos ha dado:

![primera-ip-fortigate](images/primera-ip-fortigate.png)

## Paso 2

La IP a la que tenemos que acceder es 192.168.1.99, y podemos hacer una primera prueba de conectividad:

![conectividad-ip-fortigate](images/conectividad-ip-fortigate.png)

Viendo que funciona, ya podemos acceder desde el navegador:

![primer-acceso-navegador](images/primer-acceso-navegador.png)

## Paso 3

Las primeras credenciales son las siguientes:

![primeras-credenciales](images/primeras-credenciales.png)

*(Sí, sin contraseña)*

Tras entrar nos obliga a cambiar la contraseña:

![cambio-credenciales](images/cambio-credenciales.png)

Por ejemplo le podemos poner admin también de contraseña.

## Paso 4

Nos devuelve a la pantalla de login inicial, y tras entrar con las credenciales admin admin, nos pide una pequeña configuración inicial:

![configuracion-inicial-1-fortigate](images/configuracion-inicial-1-fortigate.png)

El hostname es adecuado:

![configuracion-inicial-2-fortigate](images/configuracion-inicial-2-fortigate.png)

Con el dashboard básico nos bastaría:

![configuracion-inicial-3-fortigate](images/configuracion-inicial-3-fortigate.png)

Ya tendríamos acceso por completo al FortiGate:

![configuracion-inicial-4-fortigate](images/configuracion-inicial-4-fortigate.png)

*(Falla la carga del vídeo de introducción porque aún no tenemos configurada la salida a Internet)*
