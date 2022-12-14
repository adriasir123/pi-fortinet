---
title: "Práctica: Instalación de Debian 10"
---

# NVIDIA sistema híbrido

explicación optimus

https://www.nvidia.com/en-us/geforce/technologies/optimus/technology/

Por defecto el monitor lo controla normalmente la integrada intel, pero cada vez que se abre una aplicación, el sistema optimus averigua si esa aplicación es posible que se beneficio del rendimiento de la gráfica nvidia, y la activa. La gráfica NVIDIA pasa de estado idle a activo según se quiera usar o no.



En debian, el hecho de tener simplemente el driver de nvidia, no habilita optimus (sistema híbrido), pero sí, si instalamos bumblebee. Bumblebee es lo que habilita realmente, que este sistema híbrido entre en funcionamiento. ????PERO MIGUEL SIN BUMBLEBEE PARECE TENER UN SISTEMA HÍBRIDO TAMBIÉN


(nota: teniendo el sistema híbrido, uno puede decir si quiere usar sólo una gráfica, o la otra)



Explicación del sistema optimus en debian

https://wiki.debian.org/NVIDIA%20Optimus

Con optirun se pueden ejecutar aplicaciones diciendo que sea específicamente


En mi sistema actualmente tengo sistema híbrido, porque con lsmod aparecen los drivers nvidia+i915, qoue están siendo usados.



