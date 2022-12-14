---
title: "Práctica 4 BD: Interconexión de Servidores de Bases de Datos"
---

# Enunciado 

**Los servidores enlazados siempre tendrán que estar instalados en máquinas diferentes**

En ésta práctica veremos varias formas de crear un enlace entre distintos servidores de bases de datos

Tipos de enlace a crear:

* 2 servidores de bases de datos ORACLE
      
* 2 servidores de bases de datos PostgreSQL
      
* 1 servidor ORACLE y 1 PostgreSQL/MySQL empleando Heterogeneous Services
      
En todos ellos, es necesario explicar la configuración necesaria para ambos extremos y demostrar su funcionamiento




# 1. Registro de acciones previas en cada máquina

## Oracle1
* Añadimos lo siguiente en `/etc/apt/sources.list`
```
# Oracle XE Debian repository
deb http://oss.oracle.com/debian unstable main non-free
```
* Crea usuarios y grupos
```
sudo addgroup --system oinstall
sudo addgroup --system dba
sudo adduser --system --ingroup oinstall --shell /bin/bash oracle
sudo adduser oracle dba
```
* sysctl  
Creamos el fichero `/etc/sysctl.d/local-oracle.conf` con el siguiente contenido
```
fs.file-max = 65536
fs.aio-max-nr = 1048576
# semaphores: semmsl, semmns, semopm, semmni
kernel.sem = 250 32000 100 128
# (Oracle recommends total machine Ram -1 byte)
kernel.shmmax = 2147483648
kernel.shmall = 2097152
kernel.shmmni = 4096
net.ipv4.ip_local_port_range = 1024 65000
vm.hugetlb_shm_group = 124
vm.nr_hugepages = 64
```
> El valor de `vm.hugetlb_shm_group`, debe ser el GID del grupo dba

Para cargar ésta configuración, hacemos
```
sudo sysctl -p /etc/sysctl.d/local-oracle.conf
```

* Security limits  
Crear fichero `/etc/security/limits.d/local-oracle.conf` con el siguiente contenido
```
oracle          soft    nproc           2047
oracle          hard    nproc           16384
oracle          soft    nofile          1024
oracle          hard    nofile          65536
oracle          soft    memlock         204800
oracle          hard    memlock         204800
```

* Enlaces simbólicos requeridos (estos son los que se recomiendan crear, pero en mi caso algunos no los necesitaba, porque simplemente el binario ya existía en el directorio, o ya había un symlink creado en el mismo)
```
sudo ln -s /usr/bin/awk /bin/awk
sudo ln -s /usr/bin/basename /bin/basename
sudo ln -s /usr/bin/rpm /bin/rpm
sudo ln -s /usr/lib/x86_64-linux-gnu /usr/lib64
```

* Estructura de directorios necesaria para instalar Oracle
```
sudo mkdir -p /opt/oracle/product/12.1.0.2
sudo mkdir -p /opt/oraInventory
sudo chown -R oracle:dba /opt/oracle/
sudo chown -R oracle:dba /opt/oraInventory
```

* Pre-instalación de Oracle 12c  
Instalamos algunos paquetes necesarios
```
sudo apt install build-essential binutils libcap-dev gcc g++ libc6-dev ksh libaio-dev make libxi-dev libxtst-dev libxau-dev libxcb1-dev sysstat rpm xauth unzip
```
Además de todos esos paquetes, hay uno en específico que he necesitado porque el instalador utiliza el binario `xdpyinfo` (para comprobar cierta información acerca del DISPLAY de la máquina)
```
sudo apt install x11-utils
```

Luego cuando tengamos Oracle 12c descargado, lo descomprimimos en el home de nuestro usuario 
```
unzip linuxx64_12201_database.zip
```
Creamos una serie de variables de entorno
```
export ORACLE_HOSTNAME=localhost
export ORACLE_OWNER=oracle
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=/opt/oracle/product/12.1.0.2/dbhome_1
export ORACLE_UNQNAME=orcl
export ORACLE_SID=orcl
export PATH=$PATH:$ORACLE_HOME/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/x86_64-linux-gnu:/bin/lib:/lib/x86_64-linux-gnu/:/usr/lib64
```

* Instalación de Oracle 12c  
Debemos ejecutar el instalador como usuario "oracle". Para logearnos, primero le asignamos una contraseña (será 1234)
```
sudo passwd oracle
```
Y nos conectamos
```
su - oracle
```
Ahora sí, ejecutamos el instalador
```
/home/debian/database/runInstaller -IgnoreSysPreReqs
```

> Estamos instalando Oracle en Debian, distribución que no está certificada por Oracle. Por ello el que ejecutemos el instalador diciéndole que "ignore algunas comprobaciones" o "pre-requisitos" (usando -IgnoreSysPreReqs)

* Datos importantes introducidos durante la instalación   

**Global database name** = orcl  
**Oracle system identifier (SID)** = orcl  
**Contraseña SYS** = Megabyte1  
**Contraseña SYSTEM** = Megabyte1  






## Oracle2













## PostgreSQL1

* En el SO  
Instalamos PostgreSQL y creamos un nuevo usuario
```
sudo apt install postgresql-11
sudo adduser postgres1_user
```
La password de este usuario será 1234
```
sudo passwd postgres
```
Es necesario añadirle al usuario postgres una contraseña primero, para que nos podamos logear, y así entrar como admin a la base de datos  


* En PostgreSQL  
Como usuario postgres, creamos un nuevo usuario y su base de datos
```
CREATE USER postgres1_user WITH PASSWORD '1234';
CREATE DATABASE postgres1_db OWNER postgres1_user;
```
Nos logeamos primero como el usuario creado, y nos conectamos a su base de datos
```
su - postgres1_user
psql -d postgres1_db
```
Creamos una tabla que usaremos para probar los enlaces posteriormente
```
CREATE TABLE table1(
   id serial PRIMARY KEY,
   name VARCHAR (50),
   descrip VARCHAR (50)
);
```
Le añadimos un registro
```
INSERT INTO table1 VALUES (1, 'Bird', 'Has wings');
```

* Configuraciones para accesos remotos
  * Modificamos el fichero `/etc/postgresql/11/main/postgresql.conf`
  ```
  listen_addresses = '*'
  ```
  * Modificamos el fichero `/etc/postgresql/11/main/pg_hba.conf`. Añadí lo siguiente al final
  ```
  #Added by me
  host all all 0.0.0.0/0 md5
  ```
  * Al final, tenemos que reiniciar PostgreSQL para que los cambios surtan efecto
  ```
  sudo systemctl restart postgresql
  ```





## PostgreSQL2

* En el SO  
Instalamos PostgreSQL y creamos un nuevo usuario
```
sudo apt install postgresql-11
sudo adduser postgres2_user
```
La password de este usuario será 1234
```
sudo passwd postgres
```
Es necesario añadirle al usuario postgres una contraseña primero, para que nos podamos logear, y así entrar como admin a la base de datos  


* En PostgreSQL  
Como usuario postgres, creamos un nuevo usuario y su base de datos
```
CREATE USER postgres2_user WITH PASSWORD '1234';
CREATE DATABASE postgres2_db OWNER postgres2_user;
```
Nos logeamos primero como el usuario creado, y nos conectamos a su base de datos
```
su - postgres2_user
psql -d postgres2_db
```
Creamos una tabla que usaremos para probar los enlaces posteriormente
```
CREATE TABLE table2(
   id serial PRIMARY KEY,
   name VARCHAR (50),
   descrip VARCHAR (50)
);
```
Le añadimos un registro
```
INSERT INTO table2 VALUES (1, 'Dog', 'Has paws');
```

* Configuraciones para accesos remotos
  * Modificamos el fichero `/etc/postgresql/11/main/postgresql.conf`
  ```
  listen_addresses = '*'
  ```
  * Modificamos el fichero `/etc/postgresql/11/main/pg_hba.conf`. Añadí lo siguiente al final
  ```
  #Added by me
  host all all 0.0.0.0/0 md5
  ```
  * Al final, tenemos que reiniciar PostgreSQL para que los cambios surtan efecto
  ```
  sudo systemctl restart postgresql
  ```











# 2. Enlace ORACLE - ORACLE
















# 3. Enlace PostgreSQL - PostgreSQL













# 4. Enlace ORACLE - PostgreSQL

