# Práctica 2: Interconexión de servidores

## Pasos previos

En esta práctica voy a reutilizar el escenario de la práctica 1.

Modifico el `Vagrantfile` para añadir un segundo servidor Oracle:

![oracle1y2](https://i.imgur.com/DQqt2Y8.png)

Instalaré Oracle en el segundo servidor añadido siguiendo [mi propia documentación](https://www.servidoresclientes.ga/ri/alumno1/#1-oracle-19c) de la práctica anterior.

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

```shell
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

```shell
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

> explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.



## Oracle ⇆ PostgreSQL

> Realizar un enlace entre un servidor ORACLE y otro Postgres o MySQL empleando Heterogeneus Services, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.





