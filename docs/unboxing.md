# Unboxing

Este es un apartado que dudé en añadir ya que podría verse como innecesario, pero lo veo interesante.

Al final estoy trabajando con un producto que me han prestado, pero es como si lo hubiese comprado. Como yo lo entiendo, el proyecto no empieza cuando configuramos el firewall, el proyecto empieza desde el minuto 0 cuando estamos abriendo la caja.

Así pues, empezamos.

![caja](images/caja.jpeg)

Lo primero que nos encontramos es con una caja cuadrada muy básica, sin diseños más allá del logo de Fortinet en la cara superior.

En la cara inferior simplemente hay una etiqueta pegada con información sobre el modelo y varios códigos de barras.

![caja-abierta](images/caja-abierta.jpeg)

Nada más abrir la caja nos encontramos con un cable Ethernet. Características:

- Tipo: TIA/EIA-568-B en ambos extremos, así que es un cable directo
- Categoría: 5e (100-350MHz, 1000Mbps)
- Protección: UTP (par trenzado sin apantallar)

Es decir es así:

![TIA/EIA-568-B](images/RJ-45_TIA-568B_Right.png)

Y así:

![UTP-cable](images/UTP-cable.jpg)

![cables-sacados](images/cables-sacados.jpeg)

Aquí he sacado todos los cables juntos para que se vean mejor.

El cable de alimentación está dividido en 2 y se encontraba en los laterales de la caja. Es de modelo fsp036-rhbn3 2 pines, que irá conectado al puerto DC 12V.

![fortigate-cara-superior](images/fortigate-cara-superior.jpeg)

Pasamos al FortiGate, aquí vemos su cara superior. Tenemos algunas instrucciones de inicio.

![fortigate-cara-inferior](images/fortigate-cara-inferior.jpeg)

En la cara inferior tenemos información sobre el modelo. Nuestro dispositivo es un FortiGate 60F.
