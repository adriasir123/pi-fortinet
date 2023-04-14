# Definición de pre-proyecto

## Título

Introducción a Fortinet, ciberseguridad de última generación.

## Descripción

**¿Qué se quiere realizar? ¿Qué objetivos se quieren conseguir?**

GMV trabaja con productos muy top de ciberseguridad y maneja varios fabricantes, uno de ellos Fortinet.

La idea sería usar el FortiGate prestado durante el periodo que dure el PI, y devolverlo al finalizar el periodo.

Esto serviría para matar 2 pájaros de un tiro:

* Formación gratis para la empresa
* Me sirve para proyecto

El PI consistiría en básicamente formarme sobre todo lo pueda hacer el producto en modo local, aunque lo más probable es que no pueda formarme en lo referente a cloud ya que va licenciado y no podría acceder a un entorno de WAN.

**¿Por qué se elige este proyecto?**

Porque para mí es la idea de proyecto perfecta:

* Le sirve a la empresa
* Te sirve a tí
* Tienes un hardware sobre el que trabajar e investigar extensivamente
* Es un proyecto muy "exclusivo" digamos. Es hardware que vale MUCHO dinero, y que normalmente sólo accederías a él estando en una empresa que trabaje con dicho fabricante. Además de que el hecho de que la empresa confíe en tí y te preste el hardware ya es un punto MUY a favor.

**¿Qué problema puede solucionar?**

Según investigué en la lista de proyectos, no encontré a nadie que hubiera conseguido hardware por parte de su empresa para hacer un proyecto.

Esto lo ví como oportunidad perfecta para llevar a cabo esta iniciativa, y después de ir hablando se ha podido conseguir.

En GMV me han explicado que hoy en día un firewall no protege de nada y se necesitan muchísimos más mecanismos en entornos reales de protección. Este proyecto serviría de concienciación sobre protección real y puntera, y acercamiento a hardware y software totalmente propietario, corporativo y nuevo.

## Tecnologías a utilizar

**Hardware**: FortiGate 60F

**Proveedor cloud**: lo más probable es que no haya necesidad de usar AWS, Azure etc. Existe FortiGate Cloud pero no dispondremos de licencia para usarlo.

**Máquinas físicas**: sin contar el propio firewall, usaré mi portátil y probablemente la raspberry. No por ningún motivo en especial, sino porque son las máquinas de las que dispongo que puedo conectar físicamente al firewall.

**Máquinas virtuales**: se podría tener un pequeño escenario de máquinas en el portátil en modo bridge, 2 o 3 máquinas, para simular una red que el firewall protegería.

**Software**: con el que más se trabajará sera FortiOS, el propio sistema operativo del firewall. También se puede usar FortiClient para hacer una VPN. Para la memoria web estaremos usando mkdocs. Básicamente iremos usando todo el software que vaya siendo necesario según surja la necesidad.

## Resultados a obtener

El firewall tiene 2 modos de operación principales: local y cloud. Lógicamente el modo cloud no lo podremos usar porque no tenemos y licencia y porque no estamos poniéndolo en producción, con acceso a WANs (que es como realmente se usa este firewall en la realidad).

Igualmente, este firewall sirve para local perfectamente ya que tiene muchísimas funcionalidades, así que el resultado esperado primero de todo será obtener una memoria documentando todo lo que se puede hacer con FortiOS. Se accede normalmente desde una interfaz web, y la idea sería documentar todas o todo lo que se pueda con respecto a las opciones que tenemos con FortiOS. Preveo que no se acabe documentando la interfaz web al 100% porque como mencioné antes, hay muchas cosas de WAN que no podré probar o cosas licenciadas.

Las demos podrían ser desde mostrar varias funcionalidades generales hasta probar a hacer ataques, también tiene un puerto de DMZ donde por ejemplo podría conectar la raspberry e instalar servicios comunes de una DMZ...etc. Realmente como dije en la interfaz web ya veremos todo lo que se puede hacer, y a raíz de ahí podrían surgir todas las demos que queramos.
