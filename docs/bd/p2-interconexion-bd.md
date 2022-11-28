# Práctica 2: Interconexión de servidores

## Pasos previos

En esta práctica voy a reutilizar el escenario de la práctica 1.

Modifico el `Vagrantfile` para añadir un segundo servidor Oracle:

![oracle1y2](https://i.imgur.com/DQqt2Y8.png)

Instalaré Oracle en el segundo servidor añadido siguiendo [mi propia documentación](https://www.servidoresclientes.ga/ri/alumno1/#1-oracle-19c) de la práctica anterior.

Modifico el `Vagrantfile` para añadir un segundo servidor PostgreSQL:

![postgresql1y2](https://i.imgur.com/9oT9suI.png)

Instalaré PostgreSQL en el segundo servidor añadido siguiendo [mi propia documentación](https://www.servidoresclientes.ga/ri/alumno1/#2-postgresql) de la práctica anterior.

## Oracle ⇆ Oracle

### En `servidororacle1`

Añado lo siguiente a `/opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora`:

```shell
ORACLE2=
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.0.0.3)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = ORCLCDB)
    )
  )
```

Creo el enlace:

```sql
CREATE DATABASE LINK oracle2
CONNECT TO SCOTT IDENTIFIED BY tiger
USING 'ORACLE2';
```

Muestro la tabla `dept` remota usando el enlace:

![deptoracle2](https://i.imgur.com/yj3zKvW.png)

Compruebo que **sin usar el enlace** estoy en `servidororacle1`:

![sinenlace](https://i.imgur.com/CDDJjfU.png)

Compruebo que **usando el enlace** estoy en `servidororacle2`:

![conenlace](https://i.imgur.com/6hxgSqd.png)

### En `servidororacle2`

Añado lo siguiente a `/opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora`:

```shell
ORACLE1=
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.0.0.2)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = ORCLCDB)
    )
  )
```

Creo el enlace:

```sql
CREATE DATABASE LINK oracle1
CONNECT TO SCOTT IDENTIFIED BY tiger
USING 'ORACLE1';
```

Muestro la tabla `dept` remota usando el enlace:

![deptoracle1](https://i.imgur.com/DMwPLhl.png)

Compruebo que **sin usar el enlace** estoy en `servidororacle2`:

![sinenlace](https://i.imgur.com/FseG8QN.png)

Compruebo que **usando el enlace** estoy en `servidororacle1`:

![conenlace](https://i.imgur.com/keNoqto.png)

## PostgreSQL ⇆ PostgreSQL

### En `servidorpostgresql1`

Añado la extensión necesaria:

```sql
postgres=# create extension dblink;
CREATE EXTENSION
```

Compruebo que se ha añadido:

```shell
postgres=# \dx
                                 List of installed extensions
  Name   | Version |   Schema   |                         Description
---------+---------+------------+--------------------------------------------------------------
 dblink  | 1.2     | public     | connect to other PostgreSQL databases from within a database
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(2 rows)
```

Creo el "Foreign server" apuntando a `servidorpostgresql2`:

```sql
postgres=# CREATE SERVER servidorpostgresql2 FOREIGN DATA WRAPPER dblink_fdw OPTIONS ( host '10.0.1.3' , dbname 'bibliofilos' , port '5432');
CREATE SERVER
```

Compruebo que se ha creado:

```sql
\des+
```

```shell
                                                                     List of foreign servers
        Name         |  Owner   | Foreign-data wrapper | Access privileges | Type | Version |                     FDW options                      | Description
---------------------+----------+----------------------+-------------------+------+---------+------------------------------------------------------+-------------
 servidorpostgresql2 | postgres | dblink_fdw           |                   |      |         | (host '10.0.1.3', dbname 'bibliofilos', port '5432') |
(1 row)

(END)
```

Creo el "USER MAPPING":

```sql
postgres=# CREATE USER MAPPING FOR postgres SERVER servidorpostgresql2 OPTIONS ( user 'bibliofilos_admin' , password '1234');
CREATE USER MAPPING
```

Compruebo que se ha creado:

```sql
postgres=# \deu+
                              List of user mappings
       Server        | User name |                  FDW options
---------------------+-----------+-----------------------------------------------
 servidorpostgresql2 | postgres  | ("user" 'bibliofilos_admin', password '1234')
(1 row)
```

Pruebo la conexión:

```sql
postgres=# SELECT dblink_connect('my_new_conn1', 'servidorpostgresql2');
 dblink_connect
----------------
 OK
(1 row)
```

Muestro la tabla `bibliotecas` remota usando el enlace:

```sql
select * from dblink('servidorpostgresql2','select * from bibliotecas') as object_list(id integer, ciudad varchar, calle varchar);
```

![bibliotecaspostgres2](https://i.imgur.com/qCnaUTd.png)

Compruebo que **sin usar el enlace** estoy en `servidorpostgresql1`:

![sinenlace](https://i.imgur.com/6aG1tJh.png)

No obtengo IP y esto es correcto, ya que estoy conectado localmente usando un socket Unix, y no se usan IPs.

Compruebo que **usando el enlace** estoy en `servidorpostgresql2`:

```sql
select * from dblink('servidorpostgresql2','select inet_server_addr();') as object_list(ip varchar);
```

![conenlace](https://i.imgur.com/wg8qZZU.png)

Muestro que efectivamente esa es la IP de `servidorpostgresql2`:

![ipservidorpostgresql2](https://i.imgur.com/orCpJY7.png)

### En `servidorpostgresql2`

Añado la extensión necesaria:

```sql
postgres=# create extension dblink;
CREATE EXTENSION
```

Compruebo que se ha añadido:

```shell
postgres=# \dx
                                 List of installed extensions
  Name   | Version |   Schema   |                         Description
---------+---------+------------+--------------------------------------------------------------
 dblink  | 1.2     | public     | connect to other PostgreSQL databases from within a database
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(2 rows)
```

Creo el "Foreign server" apuntando a `servidorpostgresql1`:

```sql
postgres=# CREATE SERVER servidorpostgresql1 FOREIGN DATA WRAPPER dblink_fdw OPTIONS ( host '10.0.1.2' , dbname 'bibliofilos' , port '5432');
CREATE SERVER
```

Compruebo que se ha creado:

```sql
\des+
```

```shell
                                                                     List of foreign servers
        Name         |  Owner   | Foreign-data wrapper | Access privileges | Type | Version |                     FDW options                      | Description
---------------------+----------+----------------------+-------------------+------+---------+------------------------------------------------------+-------------
 servidorpostgresql1 | postgres | dblink_fdw           |                   |      |         | (host '10.0.1.2', dbname 'bibliofilos', port '5432') |
(1 row)

(END)
```

Creo el "USER MAPPING":

```sql
postgres=# CREATE USER MAPPING FOR postgres SERVER servidorpostgresql1 OPTIONS ( user 'bibliofilos_admin' , password '1234');
CREATE USER MAPPING
```

Compruebo que se ha creado:

```sql
postgres=# \deu+
                              List of user mappings
       Server        | User name |                  FDW options
---------------------+-----------+-----------------------------------------------
 servidorpostgresql1 | postgres  | ("user" 'bibliofilos_admin', password '1234')
(1 row)
```

Pruebo la conexión:

```sql
postgres=# SELECT dblink_connect('my_new_conn1', 'servidorpostgresql1');
 dblink_connect
----------------
 OK
(1 row)
```

Muestro la tabla `bibliotecas` remota usando el enlace:

```sql
select * from dblink('servidorpostgresql1','select * from bibliotecas') as object_list(id integer, ciudad varchar, calle varchar);
```

![bibliotecaspostgres1](https://i.imgur.com/ZxD89CN.png)

Compruebo que **sin usar el enlace** estoy en `servidorpostgresql2`:

![sinenlace](https://i.imgur.com/Ncjdype.png)

No obtengo IP y esto es correcto, ya que estoy conectado localmente usando un socket Unix, y no se usan IPs.

Compruebo que **usando el enlace** estoy en `servidorpostgresql1`:

```sql
select * from dblink('servidorpostgresql1','select inet_server_addr();') as object_list(ip varchar);
```

![conenlace](https://i.imgur.com/m1AeaH7.png)

Muestro que efectivamente esa es la IP de `servidorpostgresql1`:

![ipservidorpostgresql1](https://i.imgur.com/CIuUM5r.png)

## Oracle ⇆ PostgreSQL

### `servidororacle1` → `servidorpostgresql1`

Instalo el driver para la conexión con PostgreSQL:

```shell
sudo apt install odbc-postgresql
```

Muestro el fichero que se me ha generado:

```shell
vagrant@servidororacle1:~$ cat /etc/odbcinst.ini
[PostgreSQL ANSI]
Description=PostgreSQL ODBC driver (ANSI version)
Driver=psqlodbca.so
Setup=libodbcpsqlS.so
Debug=0
CommLog=1
UsageCount=1

[PostgreSQL Unicode]
Description=PostgreSQL ODBC driver (Unicode version)
Driver=psqlodbcw.so
Setup=libodbcpsqlS.so
Debug=0
CommLog=1
UsageCount=1
```

Dejo `odbc.ini` de la siguiente manera:

```shell
sudo nano /etc/odbc.ini
```

```shell
[postgresql]
Description = postgresql
Driver = /usr/lib/x86_64-linux-gnu/odbc/psqlodbcw.so
ServerName = 192.168.121.222
Username = bibliofilos_admin
Password = 1234
Port = 5432
Database = bibliofilos
```

Hago una prueba de conexión:

```shell
isql -v postgresql
```

![isqlpostgresql](https://i.imgur.com/dIevmYu.png)

Creo el fichero `initpostgresql.ora`:

```shell
sudo nano /opt/oracle/product/19c/dbhome_1/hs/admin/initpostgresql.ora
```

Con el siguiente contenido:

```shell
# This is a sample agent init file that contains the HS parameters that are
# needed for the Database Gateway for ODBC

#
# HS init parameters
#
HS_FDS_CONNECT_INFO = postgresql
HS_FDS_TRACE_LEVEL = 4
HS_FDS_SHAREABLE_NAME = /usr/lib/x86_64-linux-gnu/odbc/psqlodbcw.so
HS_LANGUAGE = AMERICAN_AMERICA.WE8ISO8859P9

#
# ODBC specific environment variables
#
set ODBCINI=/etc/odbc.ini

#
# Environment variables required for the non-Oracle system
#
```

Añado la siguiente entrada a `tnsnames.ora`:

```shell
sudo nano /opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora
```

```shell
postgresql =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = servidororacle1)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = postgresql)
    )
    (HS = OK)
  )
```

Dejo `listener.ora` de la siguiente manera:

```shell
sudo nano /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
```

```shell
# listener.ora Network Configuration File: /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
# Generated by Oracle configuration tools.

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = ORCLCDB)
      (ORACLE_HOME = /opt/oracle/product/19c/dbhome_1)
      (SID_NAME = ORCLCDB)
    )
    (SID_DESC =
      (SID_NAME = postgresql)
      (ORACLE_HOME = /opt/oracle/product/19c/dbhome_1)
      (PROGRAM = dg4odbc)
    )
  )
```

!!! info

    He añadido un nuevo `SID`

Reinicio el listener:

```shell
oracle@servidororacle1:~$ lsnrctl reload

LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 27-NOV-2022 23:38:45

Copyright (c) 1991, 2019, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=1521)))
The command completed successfully
```

Pruebo que funciona el nuevo registro de tns:

```shell
oracle@servidororacle1:~$ tnsping postgresql

TNS Ping Utility for Linux: Version 19.0.0.0.0 - Production on 27-NOV-2022 23:56:17

Copyright (c) 1997, 2019, Oracle.  All rights reserved.

Used parameter files:
/opt/oracle/product/19c/dbhome_1/network/admin/sqlnet.ora


Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = servidororacle1)(PORT = 1521))) (CONNECT_DATA = (SID = postgresql)) (HS = OK))
OK (0 msec)
```

Creo el enlace:

```sql
CREATE DATABASE LINK postgresql
CONNECT TO "bibliofilos_admin" IDENTIFIED BY "1234"
USING 'postgresql';
```

Muestro la tabla `libros` remota usando el enlace:

![librospostgresql1](https://i.imgur.com/eTWZjuF.png)

### `servidorpostgresql1` → `servidororacle1`

Instalo el cliente de Oracle [siguiendo mi propia documentación de la práctica 1](https://www.servidoresclientes.ga/ri/alumno1/#18-instalacion-del-cliente).

Descargo el paquete SDK del cliente de Oracle:

```shell
wget https://download.oracle.com/otn_software/linux/instantclient/218000/instantclient-sdk-linux.x64-21.8.0.0.0dbru.zip
```

Descomprimo:

```shell
sudo unzip -d /opt/oracle instantclient-sdk-linux.x64-21.8.0.0.0dbru.zip
```

Añado la siguiente variable a `.bashrc`:

```shell
sudo nano ~/.bashrc
```

```shell
export ORACLE_HOME=/opt/oracle/instantclient_21_8
```

Recargo el fichero:

```shell
source ~/.bashrc
```

Instalo los requerimientos de compilación:

```shell
sudo apt install build-essential postgresql-server-dev-13
```

Descargo el código fuente de la extensión `oracle_fdw`, descomprimo, y entro al directorio:

```shell
wget https://github.com/laurenz/oracle_fdw/archive/refs/tags/ORACLE_FDW_2_4_0.tar.gz
tar -xvzf ORACLE_FDW_2_4_0.tar.gz
cd oracle_fdw-ORACLE_FDW_2_4_0
```

Compilo e instalo:

```shell
make
sudo make install
```

Dejo `ld.so.conf` de la siguiente manera:

```shell
sudo nano /etc/ld.so.conf
```

```shell
include /etc/ld.so.conf.d/*.conf

/opt/oracle/instantclient_21_8
/usr/share/postgresql/13/extension
```

Aplico los cambios y reinicio PostgreSQL:

```shell
sudo ldconfig
sudo systemctl restart postgresql
```

Añado la extensión necesaria:

```sql
postgres=# CREATE EXTENSION oracle_fdw;
CREATE EXTENSION
```

Compruebo que se ha añadido:

```shell
postgres=# \dx
                                   List of installed extensions
    Name    | Version |   Schema   |                         Description
------------+---------+------------+--------------------------------------------------------------
 dblink     | 1.2     | public     | connect to other PostgreSQL databases from within a database
 oracle_fdw | 1.2     | public     | foreign data wrapper for Oracle access
 plpgsql    | 1.0     | pg_catalog | PL/pgSQL procedural language
(3 rows)
```

Creo el "Foreign server" apuntando a `servidororacle1`:

```sql
postgres=# CREATE SERVER servidororacle1 FOREIGN DATA WRAPPER oracle_fdw OPTIONS (dbserver '//192.168.121.211:1521/ORCLCDB');
CREATE SERVER
```

Compruebo que se ha creado:

```sql
\des+
```

```shell
                                                                     List of foreign servers
        Name         |  Owner   | Foreign-data wrapper | Access privileges | Type | Version |                     FDW options                      | Description
---------------------+----------+----------------------+-------------------+------+---------+------------------------------------------------------+-------------
 servidororacle1     | postgres | oracle_fdw           |                   |      |         | (dbserver '//192.168.121.211:1521/ORCLCDB')          |
 servidorpostgresql2 | postgres | dblink_fdw           |                   |      |         | (host '10.0.1.3', dbname 'bibliofilos', port '5432') |
(2 rows)

(END)
```

Creo el "USER MAPPING":

```sql
postgres=# CREATE USER MAPPING FOR postgres SERVER servidororacle1 OPTIONS (user 'SCOTT', password 'tiger');
CREATE USER MAPPING
```

Compruebo que se ha creado:

```sql
postgres=# \deu+
                              List of user mappings
       Server        | User name |                  FDW options
---------------------+-----------+-----------------------------------------------
 servidororacle1     | postgres  | ("user" 'SCOTT', password 'tiger')
 servidorpostgresql2 | postgres  | ("user" 'bibliofilos_admin', password '1234')
(2 rows)
```

Creo una foreign table para probar el funcionamiento:

```sql
CREATE FOREIGN TABLE DEPT (
    DEPTNO      integer OPTIONS (key 'true') NOT NULL,
    DNAME       varchar(14),
    LOC         varchar(13)
)
SERVER servidororacle1 OPTIONS (schema 'SCOTT', table 'DEPT');
```

Compruebo que se ha creado:

```sql
postgres=# select * from information_schema.foreign_tables;
 foreign_table_catalog | foreign_table_schema | foreign_table_name | foreign_server_catalog | foreign_server_name
-----------------------+----------------------+--------------------+------------------------+---------------------
 postgres              | public               | dept               | postgres               | servidororacle1
(1 row)
```

!!! info

    Una FOREIGN TABLE existe localmente en PostgreSQL pero muestra datos remotos usando un FDW. En este caso, `oracle_fdw`.

Muestro la tabla `DEPT` remota:

```sql
select * from DEPT;
```

![deptoracle1](https://i.imgur.com/f5QFfuG.png)
