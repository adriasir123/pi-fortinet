---
title: "Documentación sobre los servidores físicos"
---

# Cheatsheet básica

Estado del hardware de la máquina en general
```
ipmitool -H 172.22.221.51 -U root -P root -I lanplus chassis status -v
```

Encender la máquina
```
ipmitool -H 172.22.221.51 -U root -P root -I lanplus chassis power on
```

Apagar la máquina
```
ipmitool -H 172.22.221.51 -U root -P root -I lanplus chassis power off
```

Conectarnos por la interfaz serie redirigida de la máquina
```
ipmitool -I lanplus -H 172.22.221.51 -U root -P root sol activate
```

# ¿Qué hemos hecho en la máquina?

## Redirección de puerto COM1 (interno)

En la BIOS hemos hecho esto, y hemos redirigido su salida por el puerto ethernet IPMI.



## Interfaz IPMI

Este puerto realmente no es de red, es un puerto de administración con IP simplemente. No se puede salir de la red con él, por ejemplo.

Su configuración de IP, máscara, dirección de broadcast, etc, se hace en la BIOS. 

En mi caso, mi nodo tiene la ip 172.22.221.51



## Instalación del SO 

No lo hice yo, pero instalaron Debian 10. 

* Usuario sin privilegios
    * Nombre: usuario
    * Contraseña: usuario
* Usuario con privilegios
    * Nombre: root
    * Contraseña: root 


## Redirección de salida GRUB por COM1

Anotar cambios


## Activación de ttyS1

Es una terminal de interfaz en modo texto que ofrece el sistema operativo por el puerto COM1.

De esta manera, básicamente estamos mandando la línea de comandos a través del puerto COM1, para que así, podamos controlarlo desde la interfaz de administración.



ANOTAR CAMBIOS.

MEJORA = ACTIVACIÓN DE ESTA TERMINAL AUTOMÁTICAMENTE AL ARRANQUE MEDIANTE SYSTEMD, NO INITD 























