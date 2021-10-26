# Ejercicio 2: Control de acceso, autenticación y autorización

## ENTREGA

### Parte 1
> Dos pantallazos accediendo a departamentos.iesgn.org/intranet desde el anfitrión.

### Parte 2
> Dos pantallazos accediendo a departamentos.iesgn.org/internet desde el cliente de la red privada.

### Parte 3
> Pantallazo donde se vea la autentificación básica.

### Parte 4
> Pantallazo con las cabeceras donde se vea la autentificación básica y se vea la contraseña en clara.

### Parte 5
> Pantallazo donde se vea la autentificación digest..

### Parte 6
> Pantallazo con las cabeceras donde se vea la autentificación digest.

### Parte 7
> Pantallazos donde se comprueba el funcionamiento del ejercicio 4
























## Preliminares

### Control de acceso
> ¿Control de acceso por defecto de 000-default?

En `000-default.conf` no hay control de acceso definido, pero como el DocumentRoot está en `/var/www/html`, le afecta la siguiente directiva de `apache2.conf`:
```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```


## EJERCICIOS
### Ejercicio 0
> Definir escenario Vagrant con:
>
>- Servidor:
>    - Interfaz NAT actuando como acceso Internet
>    - Interfaz privada (veryisolated)
>- Cliente conectado a la red privada








> Crear VirtualHost `departamentos.iesgn.org`














### Ejercicio 1
> A la URL departamentos.iesgn.org/intranet sólo se debe tener acceso desde el cliente de la red local, y no se pueda acceder desde la anfitriona por la red pública. A la URL departamentos.iesgn.org/internet, sin embargo, sólo se debe tener acceso desde la anfitriona por la red pública, y no desde la red local.




### Ejercicio 2
> Autentificación básica. Limita el acceso a la URL departamentos.iesgn.org/secreto. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente. ¿Cómo se manda la contraseña entre el cliente y el servidor?.

### Ejercicio 3
> Como hemos visto la autentificación básica no es segura, modifica la autentificación para que sea del tipo digest, y sólo sea accesible a los usuarios pertenecientes al grupo directivos. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente. ¿Cómo funciona esta autentificación?


### Ejercicio 4
> Vamos a combinar el control de acceso (ejercicio 1) y la autentificación (Ejercicios 2 y 3), y vamos a configurar el virtual host para que se comporte de la siguiente manera: el acceso a la URL departamentos.iesgn.org/secreto se hace forma directa desde la intranet, desde la red pública te pide la autentificación. Muestra el resultado al profesor.
