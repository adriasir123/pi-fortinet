# Reglas de filtrado

Vamos a añadir algunas reglas que por defecto estarán deshabilitadas. Las iremos habilitando una a una para mostrar sus distintos funcionamientos.

mirar la práctica de firewall por si acaso

## Bloqueo de pings externos

Regla a crear:

![133](../images/demo/133.png)

En la lista de reglas se verá así:

![134](../images/demo/134.png)

Con esta regla, desde la red interna (donde está mi portátil), no se podrá hacer ping al exterior.

Hagamos la prueba:

![135](../images/demo/135.gif)

Como podemos ver empiezo la prueba con la regla deshabilitada, y el ping se permite.  
En cuanto que habilito la regla el ping que se estaba ejecutando deja de funcionar.  
Cuando vuelvo a deshabilitar la regla, el ping empieza a funcionar de nuevo.

##





## dns para que se haga dig a los nuestros






## ssh solo desde la red interna









