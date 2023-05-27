# Preparación de la Raspberry

## Tarjeta Micro SD

Lo primero será preparar la tarjeta, y para eso necesitaremos un adaptador USB:

![37](../images/demo/37.jpeg)

Como ya tenía datos porque había usado anteriormente la Raspberry, vamos a hacerle un formateo con GParted:

![38](../images/demo/38.png)

![39](../images/demo/39.png)

Estando ya limpia, la prepararemos con Raspberry Pi Imager. Es un programa que nos ofrece Raspberry para dejar las tarjetas listas para usar con el SO que queramos dentro del listado que nos ofrece:

![40](../images/demo/40.png)

![41](../images/demo/41.png)

![42](../images/demo/42.png)

*(Raspberry Pi OS es un derivado de Debian, así que es el SO más cercano a lo acostumbrado durante el curso, y el más estandarizado para Raspberry)*

![43](../images/demo/43.png)

![44](../images/demo/44.png)

![45](../images/demo/45.png)

![46](../images/demo/46.png)

![47](../images/demo/47.png)

Credenciales:

* Username: pi
* Password: pi

![48](../images/demo/48.png)

![49](../images/demo/49.png)

![50](../images/demo/50.png)

![51](../images/demo/51.png)

Ya sólo esperamos.

![52](../images/demo/52.png)

Ya estaría la tarjeta lista para usar, y el SO personalizado con las configuraciones que hemos decidido.

## Comprobaciones

Para mostrar que la Raspberry está funcionando, vamos a conectarla al puerto 2:

![53](../images/demo/53.jpeg)

Este **NO SERÁ SU PUERTO DEFINITIVO**, será el de la DMZ, pero para hacer pruebas de conectividad nos sirve.

Muestro las interfaces en el FortiGate para comprobar que el puerto 2 está funcionando:

![54](../images/demo/54.png)

Desde la Raspberry, podemos comprobar que le ha dado una IP con el rango correcto:

![55](../images/demo/55.jpeg)

Al estar en el mismo switch que mi portátil, tendrá conectividad con la puerta de enlace e Internet sin problemas:

![56](../images/demo/56.jpeg)

Las resoluciones DNS y el navegador también funcionan:

![57](../images/demo/57.jpeg)

## Servidor web

Es el que usaremos para las pruebas con la DMZ.

Instalamos un Apache:

```shell
sudo apt install apache2
```

Vemos que funciona:

![58](../images/demo/58.jpeg)

Cambiamos la web a algo más descriptivo:

![59](../images/demo/59.jpeg)
