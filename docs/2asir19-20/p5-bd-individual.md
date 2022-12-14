---
title: "Práctica 5 BD: Gestión del almacenamiento (individual)"
---

# Alumno 3
## Oracle

### 0. Creación del tablespace TS2 hecho por el alumno 2

* Creamos el tablespace TS2
```
create tablespace ts2
  datafile '/opt/oracle/oradata/orcl/ts2.dbf' size 1m, '/opt/oracle/oradata/ts2.dbf' size 1M
 autoextend off;
```
![](https://i.imgur.com/lBmkylT.png)


* Crear la tabla donde insertaremos registros
```
create table visitantes(
  nombre char(2000),
  edad char(2000),
  sexo char(2000),
  domicilio char(2000),
  ciudad char(2000),
  telefono char(2000)
)
TABLESPACE TS2;
```

* Los insertamos

```
insert into visitantes (nombre,edad,sexo,domicilio,ciudad,telefono)
  values ('Ana Acosta',25,'f','Avellaneda 123','Cordoba','4223344');

insert into visitantes (nombre,edad,sexo,domicilio,ciudad,telefono)
  values ('Betina Bustos',32,'fem','Bulnes 234','Cordoba','4515151');

insert into visitantes (nombre,edad,sexo,domicilio,ciudad,telefono)
  values ('Betina Bustos',32,'f','Bulnes 234','Cordoba','4515151');

insert into visitantes (nombre,edad,sexo,domicilio,ciudad,telefono)
  values ('Carlos Caseres',43,'m','Colon 345','Cordoba',03514555666);
```
> He repetido el añadido de registros tal y como hizo el alumno 2, hasta que se llenó el tablespace por completo (cosa que se requería hacer)




### 1. Muestra los objetos a los que pertenecen las extensiones del tablespace TS2 (creado por Alumno 2) y el tamaño de cada una de ellas.

```
SELECT SEGMENT_NAME,
       EXTENT_ID,
	   BYTES
  FROM DBA_EXTENTS
 WHERE TABLESPACE_NAME = 'TS2'
```



### 2. Borra la tabla que está llenando TS2 consiguiendo que vuelvan a existir extensiones libres. Añade después otro fichero de datos a TS2.

Con `truncate table` se borra una tabla, y además se libera el espacio en disco
```
TRUNCATE TABLE <nombre_tabla>;
```

Añadir datafile a tablespace
```
ALTER TABLESPACE TS2
	ADD DATAFILE '/opt/oracle/oradata/orcl/extra.dbf'; 
```


### 3. Crea el tablespace TS3 gestionado localmente con un tamaño de extensión uniforme de 128K y un fichero de datos asociado. Cambia la ubicación del fichero de datos y modifica la base de datos para que pueda acceder al mismo. Crea en TS3 dos tablas e inserta registros en las mismas. Comprueba que segmentos tiene TS3, qué extensiones tiene cada uno de ellos y en qué ficheros se encuentran.

Crear tablespace
```
CREATE TABLESPACE TS3 DATAFILE '/opt/oracle/oradata/orcl/ts3.dbf'
    EXTENT MANAGEMENT LOCAL UNIFORM SIZE 128K;
```


Mostrar los segmentos de un tablespace
```
SELECT owner, 
       segment_name,
       partition_name,
       segment_type,
       bytes
  FROM dba_segments
 WHERE tablespace_name = 'TS3'
```


Mostrar los extents que componen un segmento, junto con el fichero donde se encuentran
```
SELECT EXTENT_ID,
       FILE_ID
  FROM DBA_EXTENTS
 WHERE SEGMENT_NAME = 'nombre_del_segmento'
```





### 4. Redimensiona los ficheros asociados a los tres tablespaces que has creado de forma que ocupen el mínimo espacio posible para alojar sus objetos.

Para redimensionar un datafile:
```
alter database
datafile
   '<ruta_al_datafile>'
resize <nuevo_tamaño>;
```

Ejemplo:
```
alter database
datafile
   '/opt/oracle/oradata/orcl/ts3.dbf'
resize 50M;
```






### 5. Realiza un procedimiento llamado InformeRestricciones que reciba el nombre de una tabla y muestre los nombres de las restricciones que tiene, a qué columna o columnas afectan y en qué consisten exactamente.

### 6. Realiza un procedimiento llamado MostrarAlmacenamientoUsuario que reciba el nombre de un usuario y devuelva el espacio que ocupan sus objetos agrupando por dispositivos y archivos:

```
				Usuario: NombreUsuario

					Dispositivo:xxxx

						Archivo: xxxxxxx.xxx

								Tabla1......nnn K
								…
								TablaN......nnn K
								Indice1.....nnn K
								…
								IndiceN.....nnn K

						Total Espacio en Archivo xxxxxxx.xxx: nnnnn K

						Archivo:...
						…

				
					
					Total Espacio en Dispositivo xxxx: nnnnnn K

					Dispositivo: yyyy
					…

				Total Espacio Usuario en la BD: nnnnnnn K
```



## PostgreSQL

### 7. Averigua si es posible establecer cuotas de uso sobre los tablespaces en PostgreSQL.

Con los tablespaces de PostgreSQL, no hay manera de restringir los tamaños

[Explicación de tablespaces PostgreSQL](https://www.cybertec-postgresql.com/en/postgresql-tablespaces-its-not-so-scary/)


No hay un sistema nativo de quotas en PostgreSQL, pero hay algunas soluciones para esto

[Soluciones a las quotas en PostgreSQL](https://stackoverflow.com/questions/37822195/restrict-database-size)




## MySQL

### 8. Averigua si existe el concepto de extensión (extent) en MySQL y si coincide con el existente en ORACLE.

Sí, existe el concepto de extent, y básicamente coincide con el concepto de Oracle.

La única diferencia que podría anotar, sería más bien de nomenclatura. Con respecto a las pages en Mysql y los data blocks en Oracle, son prácticamente lo mismo, pero con distinto nombre.

En Oracle, un extent es un número específico de data blocks contiguos, y que se usan para almacenar un tipo específico de información 

En Mysql, un extent es una agrupación de pages (cada tablespace está compuesto por pages)

Los ficheros de un tablespace son llamados segments por InnoDB


Con respecto a InnoDB, es un storage engine:
>A database engine (or storage engine) is the underlying software component that a database management system (DBMS) uses to create, read, update and delete (CRUD) data from a database


[Explicación de data Blocks, extents, y segments en Oracle](https://docs.oracle.com/cd/A57673_01/DOC/server/doc/SCN73/ch3.htm)

[Explicación sobre el almacenamiento en MySQL](https://dev.mysql.com/doc/refman/8.0/en/innodb-file-space.html)

[Explicación de las pages en MySQL](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_page)


## MongoDB

### 9. Averigua si en MongoDB puede saberse el espacio disponible para almacenar nuevos documentos

En MongoDB, los datos se almacenan en forma de documentos (ficheros json), que a su vez, se agrupan en colecciones.

[Se pueden comprobar tamaño de las colecciones, almacenamiento usado en la base de datos...](https://docs.mongodb.com/manual/faq/storage/#faq-disk-size)

[Ver espacio usado en la base de datos...etc](https://stackoverflow.com/questions/9060860/how-to-check-the-available-free-space-in-mongodb)















