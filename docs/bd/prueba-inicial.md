# Prueba inicial

## Sobre Oracle

### Ejercicio 1

> Crear la tabla Alumnos de la siguiente manera:
>
> - DNI: formato 99.999.999-X
> - Nombre: inicial mayúsculas
> - Apellido: inicial mayúsculas
> - Correo: formato xxxx@xxxx.com o xxxx@xxxx.es
>
> Añadir registros

```sql
CREATE TABLE Alumnos (
    DNI char(12),
    Nombre varchar2(50),
    Apellido varchar2(50),
    Correo varchar2(50),
    CHECK (REGEXP_LIKE(DNI,'^[0-9][0-9].[0-9][0-9][0-9].[0-9][0-9][0-9]-[A-Z]$')),
    CHECK (REGEXP_LIKE(Nombre,'^[A-Z][a-z]*$')),
    CHECK (REGEXP_LIKE(Apellido,'^[A-Z][a-z]*$')),
    CHECK (REGEXP_LIKE(Correo,'*@*.com') OR REGEXP_LIKE(Correo,'*@*.es')),
    PRIMARY KEY(DNI)
);
```

![tabla alumnos](https://i.imgur.com/ssnk4aA.png)

```sql
INSERT INTO Alumnos (DNI, Nombre, Apellido, Correo)
VALUES ('12.345.678-A', 'Paco', 'Ramos', 'paco@paco.es');
INSERT INTO Alumnos (DNI, Nombre, Apellido, Correo)
VALUES ('12.345.678-B', 'Jose', 'Josefez', 'jose@jose.com');
```

![registros alumnos](https://i.imgur.com/GyWEdM2.png)

### Ejercicio 2

> Mostrar los alumnos cuyo apellido empieza por R y su correo está en un dominio .es

```sql
SELECT *
FROM Alumnos
WHERE Apellido LIKE 'R%' AND Correo LIKE '%.es';
```

![consulta alumnos](https://i.imgur.com/7zCm7XX.png)

### Ejercicio 3

> Crear la tabla Notas de la siguiente manera:
>
> - DNI: clave ajena de Alumnos
> - Módulo: ABD, SAD, Servicios o Sistemas
> - Nota: 0 al 10
>
> Añadir registros

```sql
CREATE TABLE Notas (
    DNI char(12),
    Modulo varchar2(50),
    Nota NUMBER(2),
    CONSTRAINT fk_DNI
      FOREIGN KEY (DNI)
      REFERENCES Alumnos(DNI),
    CHECK (Modulo IN ('ABD','SAD','Servicios','Sistemas')),
    CHECK (Nota BETWEEN 0 AND 10)
);
```

![tabla notas](https://i.imgur.com/YNvm9yd.png)

```sql
INSERT INTO Notas (DNI, Modulo, Nota)
VALUES ('12.345.678-A', 'ABD', '8');
INSERT INTO Notas (DNI, Modulo, Nota)
VALUES ('12.345.678-A', 'SAD', '9');
INSERT INTO Notas (DNI, Modulo, Nota)
VALUES ('12.345.678-A', 'Servicios', '10');
INSERT INTO Notas (DNI, Modulo, Nota)
VALUES ('12.345.678-A', 'Sistemas', '10');
INSERT INTO Notas (DNI, Modulo, Nota)
VALUES ('12.345.678-B', 'Servicios', '5');
```

![registros notas](https://i.imgur.com/Cmq3eMu.png)

### Ejercicio 4

> Mostrar los alumnos con más de 3 notas superiores a 7

```sql
SELECT DNI, COUNT(*) "Notas > 7"
FROM Notas
WHERE Nota > 7
GROUP BY DNI
HAVING COUNT(*) > 3;
```

### Ejercicio 5

> Hacer un procedimiento PL/SQL que:
>
> - Reciba un DNI y muestre por pantalla todas sus notas
> - Tenga las excepciones: Alumno Inexistente, Alumno sin Notas

Añado el procedimiento:

```sql
CREATE OR REPLACE PROCEDURE NOTAS_POR_DNI
(
  v_dni IN char
) AS
CURSOR c_notas IS
  SELECT * 
  FROM Notas
  WHERE DNI = v_dni;

v_reg_cursor c_notas%ROWTYPE;
v_contador1 number(1);
v_contador2 number(1);

BEGIN

  SELECT count(*)
  INTO v_contador1
  FROM Alumnos
  WHERE DNI = v_dni;

  SELECT count(*)
  INTO v_contador2
  FROM Notas
  WHERE DNI = v_dni;

  IF v_contador1 = 0 THEN
    RAISE_APPLICATION_ERROR(-20000, 'No existe un alumno con DNI ' || v_dni);
  ELSIF v_contador2 = 0 THEN
    RAISE_APPLICATION_ERROR(-20001, 'No existen notas para el alumno con DNI ' || v_dni);
  ELSE
    OPEN c_notas;
      FETCH c_notas INTO v_reg_cursor;
      WHILE c_notas%FOUND LOOP
        DBMS_OUTPUT.PUT_LINE( 'El alumno con DNI ' || v_reg_cursor.DNI || ' ha sacado una nota de ' || v_reg_cursor.Nota || ' en el módulo ' || v_reg_cursor.Modulo);
      FETCH c_notas INTO v_reg_cursor;
      END LOOP;
    CLOSE c_notas;
  END IF;

END NOTAS_POR_DNI;
/
```

![procedimiento creado](https://i.imgur.com/Sxi3fzv.png)

Activo las salidas de mensajes:

```sql
SET SERVEROUTPUT ON
```

Para ejecutar el procedimiento pidiendo el DNI tengo que ejecutar lo siguiente:

```sql
BEGIN
    NOTAS_POR_DNI('&dni');
END;
/
```

Notas de un alumno existente:

![notas A](https://i.imgur.com/SYyMGpJ.png)

Notas de un alumno inexistente:

![notas inexistente](https://i.imgur.com/RGI2n7f.png)

Alumno sin notas:

![sin notas](https://i.imgur.com/8z4v4gc.png)

## Sobre MongoDB

### Ejercicio 6

> Crear la colección `productos` con varios documentos que contengan:
>
> - Nombre
> - Precio

Creo la colección:

```sql
db.createCollection("productos")
```

![creo coleccion](https://i.imgur.com/cWakYg4.png)

La muestro:

![muestro coleccion](https://i.imgur.com/MIBzjMP.png)

Inserto los documentos:

```sql
db.productos.insertMany( [
  { nombre: "Tomate", precio: 5 },
  { nombre: "Yogurt", precio: 21 }
] )
```

![inserto documentos](https://i.imgur.com/VjvLMpV.png)

Los muestro:

![muestro documentos](https://i.imgur.com/Zznfwyt.png)

### Ejercicio 7

> Muestra los documentos correspondientes a productos cuyo precio sea superior a 20

```sql
db.productos.find(
  { precio: {$gt:20} }
)
```

![productos > 20](https://i.imgur.com/971X7j6.png)
