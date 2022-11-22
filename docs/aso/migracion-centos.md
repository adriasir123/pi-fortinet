# Migración CentOS

## Ejercicio 1

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

> Decir tu opinión

Mi opinión: han puesto fin al desarrollo de CentOS, que ahora se ha bifurcado en CentOS Linux y CentOS Stream. La última versión de CentOS Linux es la 7 que terminará su soporte en 2024, y ya no habrá más de este tipo, y se seguirá con CentOS Stream, que es la versión de desarrollo de RHEL. Esto es una mala noticia para los usuarios de CentOS, ya que no podrán seguir usando CentOS Linux, y tendrán que migrar a otra distribución, como RHEL, o a otra distribución basada en RHEL, como Oracle Linux o Rocky Linux.




En mi opinión es un movimiento sucio, pero como compraron CentOS realmente pueden hacer lo que quieran y al fin y al cabo en el mundo del software libre nada se pierde para la siempre si la gente lo quiere seguir utilizando y manteniendo, y se acabarán haciendo forks del proyecto (como ha pasado)


usados por empresas grandes incluso por amazon por ejemplo. Al volver CentOS un proyecto de upstream, de desarrollo, se han cargado la principal razón por la que la gente usaba centos, y es una táctica comercial de red hat para que la gente se pase a su sistema operativo, que es RHEL, y que es de pago.


CentOS era muy usado por grandes empresas como Amazon y Facebook, así que no les supondría un gran problema migrar a RHEL y Red Hat habría hecho un gran negocio.






## Ejercicio 2

> Crea una cuenta en Red Hat y descárgate la iso de Red Hat Enterprise Linux (RHEL) y evalúa el producto. Comenta el procedimiento de alta.














3. Descarga la iso de CentOS Stream y evalúa el producto.






4. Descarga iso de una de las otras distribuciones candidatas, indica criterios para la elección de la nueva distribución y evalúa el producto.



AlmaLinux

Rocky Linux

VZLinux

euroLinux




5. Instala CentOS 7, y evalúa la herramientas que ofrecen la distribución del punto 4.













## 


```shell
```

![]()
