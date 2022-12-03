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

> Cambia la aplicación web de forma que si se loguea el usuario raul, pueda ver los datos de la tabla o colección profesores tras los cambios de los apartados a y b

![sc6](https://i.imgur.com/TT2fKXJ.png)

![sc7](https://i.imgur.com/ODFwOqg.png)
















## Parte 2: Interconexiones

### Apartado 2A

> BD ORACLE llamada service name GN, en el esquema RAUL la tabla profesores antes creada

![sc8](https://i.imgur.com/7ouEUMq.png)

![sc9](https://i.imgur.com/NunZOmL.png)

![sc10](https://i.imgur.com/Mgg0sxK.png)

![sc11](https://i.imgur.com/FhlNurq.png)

> Crear bd en postgres llamada GN2 con tabla asignaturas

![sc12](https://i.imgur.com/G1R4zbG.png)

![sc13](https://i.imgur.com/lbdIwQW.png)

![sc14](https://i.imgur.com/dIYvFTr.png)

![sc15](https://i.imgur.com/pHNIJGl.png)

![sc16](https://i.imgur.com/B6zCaSF.png)

### Apartado 2B

> Debes realizar una consulta desde un cliente ORACLE que muestre el nombre de las asignaturas y el del profesor que las imparte usando una interconexión entre ambos servidores

![sc17](https://i.imgur.com/wTvFxSh.png)

![sc18](https://i.imgur.com/s9Hk5sT.png)

### Apartado 2C

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