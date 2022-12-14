---
title: "Tarea 7 Cloud: Hosting"
---

# Paso 0: preparativos
## Instalar/configurar proftpd
* Añadir el repositorio EPEL
No lo hago, porque ya lo tengo
* Actualizamos los paquetes (por buenas prácticas)
```
sudo yum update
```
* Instalamos el servidor FTP
```
sudo yum install proftpd
```
* Añadimos las reglas de firewall para permitir la comunicación FTP
```
sudo firewall-cmd --permanent --add-port=21/tcp
```
* Reiniciamos el servicio firewalld
```
sudo firewall-cmd --reload
```
* Por último, no nos podemos olvidar de abrir el puerto 21 en el firewall de Openstack
(esto no lo he hecho aún)



# Paso 1: Creación del usuario







# Paso 2: VirtualHost + Document Root + Default Root






# Paso 3: Crear CNAME para VH







# Paso 4: Crear usuario BD y la BD
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjExMjM0MDYwMiwtMTk1OTE4NzEzNywtMT
c3NTIxMzUxNSwtMTgyNDc1NDY0MiwxNjM3OTIwNDYxLC0xNDYz
ODMxOTksMTkyODk1MTM4MSwtMTUxNDQyNjYyMCwtMTExNTQwOD
k2NV19
-->