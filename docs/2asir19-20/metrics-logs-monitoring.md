---
title: "Práctica: Métricas, logs y monitorización"
---

# Información general

Es **necesario**:
* Realizar todos los apartados (monitorización, métricas, logs) sobre todas las máquinas: croqueta, salmorejo, tortilla y serranito
* Detallar en la documentación claramente las características de la implementación elegida, así como la forma de comprobar las características exigidas.





# 0. Pasos previos

* Creación de la nueva instancia:
    * Nombre: serranito 
    * Creada: imagen
    * OS: Debian Buster
    * Flavor: m1.mini (512 MB RAM & 10 GB disco)

* Cambios en el DNS
```
serranito-int   IN    A    10.0.0.13
serranito       IN    A    172.22.201.133
```

* Le he añadido un grupo de seguridad en Openstack a su firewall por nodo, para que permita las conexiones entrantes por el puerto 80. Es necesario hacer esto ya que Zabbix tiene un frontal web que necesita ser accesible para la instalación del mismo programa, y para su posterior uso.  

![](https://i.imgur.com/dpfY7Jd.png)








# 1. Monitorización con Zabbix
> Necesario hacer:
>* Alertas por uso excesivo de recursos (memoria, disco raíz, etc.)
>* Mostrar disponibilidad de los servicios (estado en vivo...etc)
>* Envío de alertas por correo, telegram, etc

Envío de alertas por telegram: 
https://share.zabbix.com/zabbix-tools-and-utilities/cat-notifications/zabbix-in-telegram


## Instalando Apache + php en serranito
> Sería lógico que instalásemos Zabbix en salmorejo, puesto que es donde se encuentra nuestro servidor web principal. Pero como se nos pide tener estos sistemas centralizados en serranito, vamos a tener que instalarle un servidor web


* Actualizamos la lista de paquetes, e instalamos los que no tengamos
> Hacemos esto porque serranito es una máquina nueva

```
sudo apt update
sudo apt upgrade
```

* Instalamos apache + php
```
sudo apt install apache2
sudo apt install php php-mbstring php-gd php-xml php-bcmath php-ldap php-mysql
```

* Declarando la zona horaria en PHP, en el fichero `/etc/zabbix/apache.conf`
```
php_value date.timezone Europe/Madrid 
```


## Añadiendo el repositorio de Zabbix

```
wget https://repo.zabbix.com/zabbix/4.0/debian/pool/main/z/zabbix-release/zabbix-release_4.0-3+buster_all.deb
sudo dpkg -i zabbix-release_4.0-3+buster_all.deb
```


## Instalando Zabbix server y agent en serranito

```
sudo apt update
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-agent
```

## Configuración en la base de datos

* Nos logeamos como root en tortilla
```
mysql -u root -p
```

* Creamos la base de datos, el usuario para cualquier tipo de acceso, asignamos privilegios...
```
CREATE DATABASE zabbixdb character set utf8 collate utf8_bin;
CREATE USER 'zabbix'@'%' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON zabbixdb.* TO 'zabbix'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

* Cargamos el esquema de Zabbix en la nueva base de datos creada
> Esto tendremos que hacerlo desde serranito, porque es allí donde instalamos Zabbix, y donde se encuentra el paquete que necesitamos con los datos

```
cd /usr/share/doc/zabbix-server-mysql
zcat create.sql.gz | mysql -u zabbix -p zabbixdb -h 172.22.200.158
```


## Modificar configuración de Zabbix server para la conexión con la base de datos

Editamos el fichero `/etc/zabbix/zabbix_server.conf`, modificando los siguientes parámetros
```
DBHost=10.0.0.14
DBName=zabbixdb
DBUser=zabbix
DBPassword=1234
```


## Reinicio de Apache y Zabbix

Zabbix creates its own apache configuration file /etc/zabbix/apache.conf and make a link to Apache configuration directory. Let’s use the following command to restart Apache service.

```
sudo systemctl restart apache2.service
```

Zabbix server configuration file are located at /etc/zabbix/zabbix_server.conf. Restart Zabbix server.

```
sudo systemctl restart zabbix-server
sudo systemctl restart zabbix-agent
```


## Instalación final web
> Aunque no hayamos configurado virtualhosts ni añadido datos a document roots, Zabbix funcionará para la instalación accediendo a `172.22.201.133/zabbix`. Esto se debe al enlace que hizo Zabbix a la configuración de Apache, creando un alias `/zabbix` apuntando a `/usr/share/zabbix`. Por lo tanto, éste es el document root real de la aplicación.


* Añadimos los parámetros para la conexión con la base de datos

![](https://i.imgur.com/heQIOaj.png)

> En este caso el puerto 0 está indicando que se utilizará el puerto por defecto que usa la base de datos a la que nos vamos a conectar (mariadb en nuestro caso, puerto 3306)


* Como hemos instalado Zabbix server en la misma máquina donde tenemos el frontal web, la conexión a este servicio va a ser desde localhost.  
Se rellena el puerto que usa Zabbix server por defecto, así que sólo nos quedaría nombrar la conexión, si queremos

![](https://i.imgur.com/s58PEzI.png)


* Este sería el resumen de mi instalación

![](https://i.imgur.com/XDIkejI.png)


* Después de la instalación podremos hacer login sin problemas
> Usaremos para el acceso las credenciales por defecto:  
> **Username**:  Admin  
> **Password**:  zabbix
  

![](https://i.imgur.com/ciWiVEH.png)

* Una vez dentro, nos encontraremos con el siguiente panel principal. Viendo esto podremos estar seguros de que la aplicación web Zabbix se instaló y configuró correctamente

![](https://i.imgur.com/4XNIA2W.png)



## Instalando Zabbix Agent en las demás máquinas a monitorizar
### En croqueta

* Añadiendo los repositorios de Zabbix para Debian 10
```
wget https://repo.zabbix.com/zabbix/4.0/debian/pool/main/z/zabbix-release/zabbix-release_4.0-3+buster_all.deb
sudo dpkg -i zabbix-release_4.0-3+buster_all.deb
```


* Actualizamos la lista de paquetes y lo instalamos a la última versión posible
```
sudo apt update
sudo apt install zabbix-agent
```


* Configurando el fichero `/etc/zabbix/zabbix_agentd.conf` de Zabbix Agent
```
Server=10.0.0.13
Hostname=croqueta
```

> **Server**: IP apuntando a donde esté nuestro Zabbix Server (apunta a serranito)  
**Hostname**: nombre que queramos que tenga este cliente de Zabbix a la hora de monitorizarlo 


* Habilitamos siempre al arranque Zabbix Agent, y lo iniciamos 
```
sudo systemctl enable zabbix-agent 
sudo systemctl start zabbix-agent 
```


### En salmorejo








### En tortilla






















## Añadiendo los hosts en Zabbix Server para su monitorización
### Croqueta

* En el panel principal debemos irnos a Configuration / Hosts / Create host. Estaremos en la siguiente pantalla

![](https://i.imgur.com/J9xhKcF.png)

* Aquí tendremos que rellenar los datos del host que queramos añadir (en este caso croqueta), para configurar su conexión con Zabbix Server

![](https://i.imgur.com/uatvzOb.png)

* Después de ésto, asociamos una plantilla al host

![](https://i.imgur.com/6A6kerl.png)

> Las **plantillas** son conjuntos de entidades que pueden ser asignadas a varios hosts simultáneamente.  
Las **entidades** podríamos entenderlas como "características" que estamos queriendo monitorizar sobre los hosts, entre otras cosas.  
Haz click [aquí](https://www.zabbix.com/documentation/2.0/manual/config/templates) para más info

* Finalmente, solamente tenemos que añadir el host, y veremos que está en la lista. Vemos que aparece croqueta

![](https://i.imgur.com/j6JR3F5.png)







### Salmorejo










### Tortilla




















































# 2. Recolección de métricas con cacti
> A hacer en esta parte: recolección, gestión centralizada, filtrado y representación gráfica de los parámetros que consideres necesarios



-Recolección de métricas (recolección de datos simplemente)




# 3. Logs con graylog
> A hacer en esta parte: Implementa un sistema centralizado de filtrado que permita quedarse con registros con prioridad error, critical, alert o emergency. Representa gráficamente los datos relevantes extraidos de los logs o configura el envío por correo al administrador de los logs relevantes (una opción o ambas)



-Logs (centralización de logs)





# Fuentes
https://tecadmin.net/install-zabbix-on-debian/
https://tecadmin.net/install-zabbix-agent-on-ubuntu-and-debian/
https://tecadmin.net/add-host-zabbix-server-monitor/




