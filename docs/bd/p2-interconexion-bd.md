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

> Realizar un enlace entre un servidor ORACLE y otro Postgres o MySQL empleando Heterogeneus Services, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.





