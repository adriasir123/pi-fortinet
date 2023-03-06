# Migración CentOS

## Ejercicio 1

### 1.1

> Analizar el desencadenante de la retirada de CentOS 8 del mercado

CentOS era un rebuild de RHEL 100% open source, gratuito y de comunidad. Es decir, que por cada release de RHEL se lanzaba una versión de CentOS, que además solía tener en torno a 10 años de soporte. Se convirtió en unas de las distribuciones más estables junto con Debian.

En 2014 Red Hat compró CentOS pero prometió continuar con el proyecto. Este hecho empezó generar sospechas en la comunidad, ya que es muy curioso que una empresa que vende RHEL compre una distribución que es un rebuild de la misma.

Todo se hizo aparente en 2020 cuando Red Hat anunció por el [siguiente comunicado](https://lists.centos.org/pipermail/centos-announce/2020-December/048208.html) que reduciría el período de soporte de CentOS 8 de 10 años a 2, acabando el 31 de Diciembre de 2021. CentOS 8 llevaba ya 1 año de desarrollo, así que sólo dejaron otro año para que la gente pensara qué hacer. Esto creó una situación peculiar donde los que habían hecho lo correcto, actualizar a CentOS 8, se vieron obligados a plantear una migración en muy poco tiempo, y los que seguían con CentOS 7 mantendrían su soporte completo hasta el 30 de Junio de 2024.

¿Qué fue el desencadenante de esta decisión?

Como siempre, las decisiones a gran escala como esta no parten de un solo motivo sino de muchos, y de muchas situaciones en conjunto. Aún así, podemos explicar qué ha pasado de manera resumida.

CentOS siempre ha ofrecido una solución a nivel empresarial gratuita y de comunidad. Esto permitía a las empresas que no tenían presupuesto para pagar por RHEL, o que no querían pagar por RHEL, tener una alternativa de calidad y con soporte. Podían haber muchos motivos por los que las empresas y desarrolladores no quisieran pagar por RHEL, como por ejemplo ser suficientemente competentes para no necesitar soporte, o simplemente ahorrar dinero.

Tener a CentOS de esta manera para Red Hat no era de mucha utilidad, ya que objetivamente les hacía perder dinero. Al haber comprado CentOS en su momento, ahora mantenerlo claramente les generaba un gasto, gasto que además no podían recuperar ya que CentOS es totalmente gratuito. Otro dato es que obviamente esto generaba una fragmentación de la comunidad, que además no tenía ningún incentivo para migrar a RHEL, lo cual es el principal objetivo para Red Hat. También se mencionó en una entrevista que entre las comunidades de Red Hat y Fedora por ejemplo, había un involucramiento muy bidireccional, pero que desafortunadamente con CentOS no era así.

Para terminar y dar más perspectiva al asunto, hay que explicar que Red Hat tiene objetivos importantes a corto y largo plazo detrás de esta decisión. Según se dijo en una entrevista, CentOS Stream será el sucesor de CentOS Linux pero no un reemplazo, ya que la filosofía cambia completamente. El enfoque de CentOS Stream será la innovación en el linux empresarial, es decir, no será una distribución estable como lo llevaba siendo desde hace años, sino que será una distribución de rápidas actualizaciones para empresas centradas en cloud y contenedores.

Como hemos podido ver, no hay buenos ni malos en esta historia, puesto que Red Hat tiene sus motivos para hacer lo que ha hecho, y la comunidad de CentOS también tienen sus motivos para haber reaccionado de manera negativa.

### 1.2

> Decir tu opinión

Mi opinión es bastante neutra, ya que puedo entender ambos puntos de vista.

Red Hat tenía que mantener algo que no les devolvía beneficios, y encima no les era de utilidad. Ahora por lo menos le darán una nueva utilidad a CentOS que les servirá para tener ideas que implementar en futuras versiones de RHEL, su produto principal. Es decir, CentOS Stream ahora nutrirá directamente a RHEL, cosa que antes no pasaba, RHEL nutría a CentOS. Además ha sido una gran táctica comercial para obligar a las empresas a pasarse a RHEL.

Por otro lado, la comunidad de CentOS tenía una buena distribución que se les ha quitado de manera brusca, además de haber sido un movimiento bastante sucio por parte de Red Hat. Ellos prometían en torno a 10 años de soporte para CentOS 8 como venía siendo de costumbre en las releases de CentOS, pero lo redujeron a 2 años con solo 1 año de margen para que la gente se planteara la migración. Hay que tener en cuenta que este problema ha sido más grande de lo que en principio parece, ya que CentOS era muy usado por grandes empresas como Amazon y Facebook. Al transformarlo en un proyecto de upstream, de desarrollo, han eliminado la principal razón por la que la gente usaba CentOS, y es la gran estabilidad que aportaba.

La buena noticia es que estamos en el mundo del software libre, y algo no muere si la comunidad no quiere. Se han acabado haciendo forks del proyecto, como era de esperar, y se han creado nuevas distribuciones que mantienen la filosofía de CentOS, como Rocky Linux y AlmaLinux.

## Ejercicio 2

### 2.1

> Crear una cuenta en Red Hat y descargar la iso de RHEL

<https://developers.redhat.com/products/rhel/download>

![redhat1](https://i.imgur.com/rRF5O1s.png)

![redhat2](https://i.imgur.com/o86bxE4.png)

![redhat3](https://i.imgur.com/PRYaDw4.png)

![redhat4](https://i.imgur.com/jMIkKU5.png)

![redhat5](https://i.imgur.com/XvRKxvD.png)

![redhat6](https://i.imgur.com/nGM7YnJ.png)

![redhat7](https://i.imgur.com/EM3l7jA.png)

![redhat8](https://i.imgur.com/0e3NY0S.png)

### 2.2

> Evaluar el producto

Para evaluarlo, vamos a instalarlo desde la imagen descargada.

Así queda el resumen de características que le voy a dar a la VM:

![instalacion1](https://i.imgur.com/1gb4WKO.png)

Después de esto, empezamos la instalación real:

![instalacion2](https://i.imgur.com/STfFnQs.png)

![instalacion3](https://i.imgur.com/lheh3C4.png)

En el siguiente menú, los apartados marcados en rojo son los que he tenido que configurar/cambiar *(ambas contraseñas son 1234 para root y para el usuario)*:

![instalacion4](https://i.imgur.com/bwXNjQd.png)

![instalacion5](https://i.imgur.com/5FTq2RR.png)

Ya lo tenemos instalado, y muestro algunas pruebas:

![instalacion6](https://i.imgur.com/jygB4OU.png)

Dejo por aquí [un buen artículo](https://www.educba.com/redhat-vs-debian/) que compara RHEL con Debian. Está un poco desactualizado con respecto a las versiones pero por lo demás lo he encontrado interesante.

## Ejercicio 3

### 3.1

> Descargar la iso de CentOS Stream

![descargastream](https://i.imgur.com/OfiwhVM.png)

### 3.2

> Evaluar el producto

Para evaluarlo, vamos a instalarlo desde la imagen descargada.

Así queda el resumen de características que le voy a dar a la VM:

![instalacion1](https://i.imgur.com/b99Wyvr.png)

Después de esto, empezamos la instalación real:

![instalacion2](https://i.imgur.com/DdxXgNY.png)

![instalacion3](https://i.imgur.com/pVZW1iO.png)

En el siguiente menú, los apartados marcados en rojo son los que he tenido que configurar/cambiar *(ambas contraseñas son 1234 para root y para el usuario)*:

![instalacion4](https://i.imgur.com/394Q0ww.png)

![instalacion5](https://i.imgur.com/C2YL1oe.png)

Ya lo tenemos instalado, y muestro algunas pruebas:

![instalacion6](https://i.imgur.com/C6zS4pl.png)

Como hemos podido ver, desde la adquisición de Centos por parte de Red Hat, lo han dejado muy copia de RHEL, incluso el fichero para comprobar la versión es el mismo.

Red Hat [en este enlace](https://www.redhat.com/en/topics/linux/what-is-centos-stream) explica muy bien las ventajas de CentOS Stream.

## Ejercicio 4

### 4.1

> Las otras distribuciones alternativas son las siguientes:
>
>- AlmaLinux
>- Rocky Linux
>- VzLinux
>- EuroLinux
>
> Elegir una, indicar criterios para la elección, y descargar la iso

Elijo Rocky Linux ya que en mi opinión es la evolución lógica después del problema que ha habido con CentOS.

Gregory Kurtzer, el fundador de CentOS, creó el proyecto "Rocky Linux" para continuar con los ideales originales de CentOS.

Descargo la iso:

![rockydescarga](https://i.imgur.com/6LJNWh7.png)

### 4.2

> Evaluar el producto

Para evaluarlo, vamos a instalarlo desde la imagen descargada.

Así queda el resumen de características que le voy a dar a la VM:

![instalacion1](https://i.imgur.com/S6Jf9of.png)

Como Rocky es un proyecto relativamente nuevo, he tenido que seleccionar RHEL 9 en virt manager ya que es lo que más se le asemeja, puesto que Rocky aún no aparece en la lista de SSOO.

Después de esto, empezamos la instalación real:

![instalacion2](https://i.imgur.com/wIeTtie.png)

![instalacion3](https://i.imgur.com/hu99ndP.png)

En el siguiente menú, los apartados marcados en rojo son los que he tenido que configurar/cambiar *(ambas contraseñas son 1234 para root y para el usuario)*:

![instalacion4](https://i.imgur.com/fbaSG5c.png)

![instalacion5](https://i.imgur.com/6jpFnuZ.png)

Ya lo tenemos instalado, y muestro algunas pruebas:

![instalacion6](https://i.imgur.com/kA8z3Ga.png)

Como hemos podido ver, ya que Rocky parte de RHEL, pasa un poco como con CentOS. Es una copia del RHEL actual pero siguiendo la filosofía de CentOS original.

Rocky Linux 9 tendrá soporte hasta 2032, manteniendo la promesa de 10 años de soporte que se venía teniendo tradicionalmente.

[En este artículo](https://www.hostinger.com/tutorials/rocky-linux-review) describen muy bien Rocky Linux.

A destacar positivamente, se usa LVM y XFS por defecto:

![positivorockylinux](https://i.imgur.com/kLiKm36.png)

La única desventaja que le veo a Rocky Linux es que por razones obvias, aunque parta de RHEL, las características propietarias y de pago no estarán disponibles.  

Por ejemplo, el "live kernel patching" y "Red Hat Sattelite" no estarán disponibles.

## Ejercicio 5

> Instalar CentOS 7, y evalúa la herramientas que ofrecen la distribución del punto 4.














