# Examen prácticas 1 y 2

## Parte 1: App Web sobre MongoDB

### 1A

> Crear una bd llamada `GN`

![dbgn](https://i.imgur.com/U1sMALP.png)

> Crear el usuario `raul` con permisos sobre GN

![raulmongo](https://i.imgur.com/8NBLxMR.png)

### 1B

> Arrancar `MongoDB-PHP-GUI`

```shell
sudo docker run --add-host localhost:0.0.0.0 --publish 5000:5000 --rm samueltallet/mongodb-php-gui
```

> Login

![loginmongodb-php-gui](https://i.imgur.com/CtLObCW.jpg)

> Crear colección `profesores`

![profesorescol](https://i.imgur.com/7y0Mbbr.png)

> Insertar los documentos

![docsinsert](https://i.imgur.com/2pm415B.png)

### 1C

> Modificar la app web para loguear con el usuario `raul`, y ver los datos de la colección `profesores`

![loginapp](https://i.imgur.com/TT2fKXJ.png)

![contenidoapp](https://i.imgur.com/ODFwOqg.png)

!!! info

    Durante el examen se ha creado la rama `examen`, que es donde se alojan estos cambios. La app original sigue estando intacta en `main`

!!! warning

    La app internamente se sigue logueando con admin, los usuarios que escribimos no "pertenecen a la base de datos". Son usuarios ficticios digamos.  
    Sería interesante cambiar esto en un futuro














## Parte 2: Interconexiones

### 2A

#### En `servidororacle1`

> Conectar como `sys`

```shell
sqlplus / as sysdba
```

> Crear el usuario `raul` con privilegios

```sql
SQL>
create user raul identified by 1234;

User created.

grant all privileges to raul;

Grant succeeded.
```

> Cambiar a este usuario

```sql
connect raul/1234
```

> Crear la tabla `profesores`

```sql
CREATE TABLE profesores (
  DNI NUMBER(10) NOT NULL,
  Nombre VARCHAR2(50) NOT NULL,
  CONSTRAINT profesores_pk PRIMARY KEY (DNI)
);

Table created.
```

> Insertar registros

```sql
INSERT INTO profesores VALUES (28888888, 'Raul Ruiz Padilla');

1 row created.

INSERT INTO profesores VALUES (27777777, 'Rafael Luengo Sanz');

1 row created.
```

> Comprobar que se han insertado

```sql
SELECT * FROM profesores;

       DNI NOMBRE
---------- --------------------------------------------------
  28888888 Raul Ruiz Padilla
  27777777 Rafael Luengo Sanz
```

> Mostrar el `SERVICE_NAME` actual

```sql
show parameters service_name

NAME		                             TYPE	 VALUE
------------------------------------ ----------- ------------------------------
service_names		                     string	 ORCLCDB
```

> Cambiarlo a `GN`

```sql
alter system set service_names = 'GN' scope = spfile;

System altered.
```

> Reiniciar la bd

```sql
SHUTDOWN IMMEDIATE
Database closed.
Database dismounted.
ORACLE instance shut down.
STARTUP
ORACLE instance started.

Total System Global Area 1258287544 bytes
Fixed Size	            9134520 bytes
Variable Size	          788529152 bytes
Database Buffers	  452984832 bytes
Redo Buffers	            7639040 bytes
Database mounted.
Database opened.
```

> Comprobar que el `SERVICE_NAME` ha cambiado

```sql
show parameters service_name

NAME		                             TYPE	 VALUE
------------------------------------ ----------- ------------------------------
service_names		                     string	 GN
```

> Modificar el fragmento correspondiente en `tnsnames.ora`

```shell
nano /opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora
```

```shell
GN=
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = servidororacle1)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = GN)
    )
  )
```

#### En `servidorpostgresql1`

> Crear una bd llamada `GN2`

```shell
postgres=# CREATE DATABASE GN2;
CREATE DATABASE
postgres=# \l
                                   List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |       Access privileges
-------------+----------+----------+---------+---------+--------------------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres                  +
             |          |          |         |         | postgres=CTc/postgres         +
             |          |          |         |         | bibliofilos_admin=CTc/postgres
 gn2         | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 pepino      | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
(6 rows)
```

> Cambiar a la bd creada

```shell
postgres=# \connect gn2
You are now connected to database "gn2" as user "postgres".
gn2=#
```

> Crear la tabla `asignaturas` y comprobar que se ha creado

```sql
gn2=# CREATE TABLE asignaturas (
  Nombre VARCHAR(3) PRIMARY KEY,
  DNIprofesor integer NOT NULL
);
CREATE TABLE
gn2=# \dt
            List of relations
 Schema |    Name     | Type  |  Owner
--------+-------------+-------+----------
 public | asignaturas | table | postgres
(1 row)
```

> Insertar registros

```sql
gn2=# INSERT INTO asignaturas VALUES ('ASO', 28888888);
INSERT 0 1
gn2=# INSERT INTO asignaturas VALUES ('ABD', 27777777);
INSERT 0 1
```

> Comprobar que se han insertado

```sql
gn2=# SELECT * FROM asignaturas;
 nombre | dniprofesor
--------+-------------
 ASO    |    28888888
 ABD    |    27777777
(2 rows)
```

> Crear el usuario `raul` y comprobar que se ha creado

```shell
vagrant@servidorpostgresql1:~$ sudo adduser raul
Adding user `raul' ...
Adding new group `raul' (1002) ...
Adding new user `raul' (1002) with group `raul' ...
Creating home directory `/home/raul' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for raul
Enter the new value, or press ENTER for the default
	Full Name []:
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y
vagrant@servidorpostgresql1:~$ su - postgres
Password:
postgres@servidorpostgresql1:~$ psql
psql (13.8 (Debian 13.8-0+deb11u1))
Type "help" for help.

postgres=# create user raul with encrypted password '1234';
CREATE ROLE
postgres=# \du
                                       List of roles
     Role name     |                         Attributes                         | Member of
-------------------+------------------------------------------------------------+-----------
 bibliofilos_admin |                                                            | {}
 postgres          | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 raul              |                                                            | {}
```

> Darle los permisos necesarios

```shell
postgres=# GRANT all privileges ON DATABASE gn2 TO raul;
GRANT
postgres=# \l
                                   List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |       Access privileges
-------------+----------+----------+---------+---------+--------------------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres                  +
             |          |          |         |         | postgres=CTc/postgres         +
             |          |          |         |         | bibliofilos_admin=CTc/postgres
 gn2         | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres                  +
             |          |          |         |         | postgres=CTc/postgres         +
             |          |          |         |         | raul=CTc/postgres
 pepino      | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
(6 rows)

postgres=# \connect gn2
You are now connected to database "gn2" as user "postgres".
gn2=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO raul;
GRANT
gn2=# SELECT * from information_schema.table_privileges WHERE grantee = 'raul';
 grantor  | grantee | table_catalog | table_schema | table_name  | privilege_type | is_grantable | with_hierarchy
----------+---------+---------------+--------------+-------------+----------------+--------------+----------------
 postgres | raul    | gn2           | public       | asignaturas | INSERT         | NO           | NO
 postgres | raul    | gn2           | public       | asignaturas | SELECT         | NO           | YES
 postgres | raul    | gn2           | public       | asignaturas | UPDATE         | NO           | NO
 postgres | raul    | gn2           | public       | asignaturas | DELETE         | NO           | NO
 postgres | raul    | gn2           | public       | asignaturas | TRUNCATE       | NO           | NO
 postgres | raul    | gn2           | public       | asignaturas | REFERENCES     | NO           | NO
 postgres | raul    | gn2           | public       | asignaturas | TRIGGER        | NO           | NO
(7 rows)
```

### 2B `servidororacle1` → `servidorpostgresql1`

> Dejar `odbc.ini` de la siguiente manera:

```shell
sudo nano /etc/odbc.ini
```

```shell
[postgresql]
Description = postgresql
Driver = /usr/lib/x86_64-linux-gnu/odbc/psqlodbcw.so
ServerName = 192.168.121.222
Username = raul
Password = 1234
Port = 5432
Database = gn2
```

> Conectar a la bd

```shell
sqlplus / as sysdba
```

> Crear un nuevo enlace

```sql
CREATE PUBLIC DATABASE LINK postgresqlexamen
CONNECT TO "raul" IDENTIFIED BY "1234"
USING 'postgresql';
```

> Hacer una primera prueba mostrando la tabla remota `asignaturas` usando el enlace

```sql
SELECT * FROM "asignaturas"@postgresqlexamen;

nombre		                             dniprofesor
------------------------------------ -----------
ASO			                                28888888
ABD			                                27777777
```

> Hacer la consulta final usando el enlace, mostrando el nombre de las asignaturas junto con el nombre del profesor que las imparte

Consulta:

```sql
SELECT a."nombre", p.nombre
FROM "asignaturas"@postgresqlexamen a 
INNER JOIN profesores p
ON a."dniprofesor" = p.dni;
```

Resultado:

```sql
nombre		                             NOMBRE
------------------------------------ --------------------------------------------------
ASO		                             Raul Ruiz Padilla
ABD		                             Rafael Luengo Sanz
```

### 2C `servidorpostgresql1` → `servidororacle1`

> Cambiar `ORCLCDB` por `GN` en `tnsnames.ora`

```shell
sudo nano /opt/oracle/instantclient_21_8/network/admin/tnsnames.ora
```

```shell
GN=
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.121.211)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = GN)
    )
  )
```

> Conectar como `postgres` a `gn2`:

```shell
vagrant@servidorpostgresql1:~$ su - postgres
Password:
postgres@servidorpostgresql1:~$ psql gn2
psql (13.8 (Debian 13.8-0+deb11u1))
Type "help" for help.

gn2=#
```

> Añadir la extensión `oracle_fdw`

```shell
gn2=# CREATE EXTENSION oracle_fdw;
CREATE EXTENSION
```

Compruebo que se ha añadido:

```shell
gn2=# \dx
                        List of installed extensions
    Name    | Version |   Schema   |              Description
------------+---------+------------+----------------------------------------
 oracle_fdw | 1.2     | public     | foreign data wrapper for Oracle access
 plpgsql    | 1.0     | pg_catalog | PL/pgSQL procedural language
(2 rows)
```

> Crear el "foreign server" apuntando a `servidororacle1`

```sql
gn2=# CREATE SERVER servidororacle1examen FOREIGN DATA WRAPPER oracle_fdw OPTIONS (dbserver '//192.168.121.211:1521/GN');
CREATE SERVER
```

Compruebo que se ha creado:

```shell
gn2=# \des+
                                                               List of foreign servers
         Name          |  Owner   | Foreign-data wrapper | Access privileges | Type | Version |              FDW options               | Description
-----------------------+----------+----------------------+-------------------+------+---------+----------------------------------------+-------------
 servidororacle1examen | postgres | oracle_fdw           |                   |      |         | (dbserver '//192.168.121.211:1521/GN') |
(1 row)
```

Le doy permisos de uso a `raul`:

```sql
gn2=# GRANT USAGE ON FOREIGN SERVER servidororacle1examen TO raul;
GRANT
```

> Crear los "USER MAPPING"

```sql
gn2=# CREATE USER MAPPING FOR postgres SERVER servidororacle1examen OPTIONS (user 'raul', password '1234');
CREATE USER MAPPING
gn2=# CREATE USER MAPPING FOR raul SERVER servidororacle1examen OPTIONS (user 'raul', password '1234');
CREATE USER MAPPING
```

Compruebo que se han creado:

```shell
gn2=# \deu+
                        List of user mappings
        Server         | User name |           FDW options
-----------------------+-----------+----------------------------------
 servidororacle1examen | postgres  | ("user" 'raul', password '1234')
 servidororacle1examen | raul      | ("user" 'raul', password '1234')
(2 rows)
```

> Crear la "foreign table" `profesores`

```sql
CREATE FOREIGN TABLE PROFESORES (
    DNI         integer OPTIONS (key 'true') NOT NULL,
    NOMBRE      varchar(50) NOT NULL
)
SERVER servidororacle1examen OPTIONS (schema 'RAUL', table 'PROFESORES');
```

Compruebo que se ha creado:

```sql
gn2=# select * from information_schema.foreign_tables;
 foreign_table_catalog | foreign_table_schema | foreign_table_name | foreign_server_catalog |  foreign_server_name
-----------------------+----------------------+--------------------+------------------------+-----------------------
 gn2                   | public               | profesores         | gn2                    | servidororacle1examen
(1 row)
```

Le doy permisos a `raul` sobre la tabla:

```sql
gn2=# grant all on table profesores to raul;
GRANT
```

> Cambiar al usuario `raul`

```shell
gn2=# set role raul;
SET
```

Comprobar el cambio de usuario:

```shell
gn2=> select current_user, session_user;
 current_user | session_user
--------------+--------------
 raul         | postgres
(1 row)
```

> Mostrar la tabla `profesores` remota para hacer una primera prueba

```sql
gn2=> SELECT * FROM profesores;
   dni    |       nombre
----------+--------------------
 28888888 | Raul Ruiz Padilla
 27777777 | Rafael Luengo Sanz
(2 rows)
```

> Hacer la consulta final usando la "foreign table", mostrando el nombre de las asignaturas junto con el nombre del profesor que las imparte

Consulta:

```sql
SELECT a.nombre, p.nombre
FROM asignaturas a 
INNER JOIN profesores p
ON a.dniprofesor = p.dni;
```

Resultado:

```sql
 nombre |       nombre
--------+--------------------
 ASO    | Raul Ruiz Padilla
 ABD    | Rafael Luengo Sanz
(2 rows)
```
