# Web Filter

Todos los Fortigate tienen los llamados "Security Profiles", que no son más que filtros que podemos aplicar en cada una de las políticas de firewall que tengamos.

Vienen ya creados por defecto, y son los siguientes:

![144](../images/demo/144.png)

Cada uno ya viene totalmente configurado, aunque por supuesto son modificables.

¿Cómo los aplicamos?

Primero, tendremos que decidir sobre qué política de firewall aplicamos el filtro.

Siguiendo la lógica de que queremos proteger nuestra red interna, podemos aplicarlo sobre la política de salida a Internet desde la red interna:

![145](../images/demo/145.png)

Una vez dentro, sólo tendremos que marcar el tick y guardar:

![146](../images/demo/146.png)

En la lista de reglas, nos daremos cuenta de que aparece el nuevo perfil:

![147](../images/demo/147.png)

Este perfil bloqueará el tráfico web según si está permitido o no, como podemos demostrar:

![148](../images/demo/148.gif)

Como vemos, mientras que está el perfil aplicado no funciona la navegación web a un sitio de bitcoin que tiene específicamente bloqueado como bitcoin.cz.

Sin embargo cuando quitamos el perfil, vuelve a funcionar el acceso.
