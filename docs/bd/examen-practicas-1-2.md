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


























> Crear bd en postgres llamada GN2 con tabla asignaturas

![sc12](https://i.imgur.com/G1R4zbG.png)

![sc13](https://i.imgur.com/lbdIwQW.png)

![sc14](https://i.imgur.com/dIYvFTr.png)

![sc15](https://i.imgur.com/pHNIJGl.png)

![sc16](https://i.imgur.com/B6zCaSF.png)






















### 2B

> Debes realizar una consulta desde un cliente ORACLE que muestre el nombre de las asignaturas y el del profesor que las imparte usando una interconexión entre ambos servidores

![sc17](https://i.imgur.com/wTvFxSh.png)

![sc18](https://i.imgur.com/s9Hk5sT.png)

### 2C

> Debes realizar una consulta desde un cliente Postgres que muestre el nombre de las asignaturas y el del profesor que las imparte usando una interconexión entre ambos servidores

![sc19](https://i.imgur.com/wVhohqs.png)

CREATE FOREIGN TABLE profesores (
    DNI         integer,
    NOMBRE      varchar(50) NOT NULL
)
SERVER servidororacle1 OPTIONS (schema 'raul', table 'profesores');

Todo el proceso de interconexión se hace conectado a la base de datos GN2

SELECT *
FROM oracle.profesores p, asignaturas a
JOIN asignaturas a
ON (p.dni = a.dniprofesor);

select p.nombre, a.nombre
from oracle.profesores p inner join
asignaturas a on p.dni = a.dniprofesor;


CREATE TABLE asignaturas (
  Nombre VARCHAR ( 50 ) PRIMARY KEY,
  DNIProfesor serial NOT NULL
);

INSERT INTO asignaturas (Nombre, DNIProfesor) VALUES('ASO', 28888888);
INSERT INTO asignaturas (Nombre, DNIProfesor) VALUES('ABD', 27777777);



CREATE TABLE profesores (
  DNI VARCHAR2(10) NOT NULL,
  Nombre VARCHAR2(50) NOT NULL,
  CONSTRAINT profesores_pk PRIMARY KEY (DNI)
);