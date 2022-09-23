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

> Hacer un procedimiento PL/SQL que reciba un DNI y muestre por pantalla todas sus notas. Debes controlar las siguientes excepciones: Alumno Inexistente, Alumno sin Notas.

```sql
CREATE OR REPLACE PROCEDURE NOTAS_POR_DNI 
(
  v_dni IN char
) AS 
BEGIN
  SELECT Modulo, Nota INTO 
  FROM Notas
  WHERE DNI = v_dni;
END NOTAS_POR_DNI;
/

SET SERVEROUTPUT ON
BEGIN
    NOTAS_POR_DNI('&dni');
END;






CREATE OR REPLACE PROCEDURE NOTAS_POR_DNI AS
CURSOR c_empleados IS
SELECT Modulo, Nota INTO 
FROM Notas
WHERE DNI = v_dni;

v_reg_cursor c_empleados%ROWTYPE;

BEGIN
  OPEN c_empleados;
  FETCH c_empleados INTO v_reg_cursor;
  WHILE c_empleados%FOUND LOOP
    DBMS_OUTPUT.PUT_LINE(v_reg_cursor.first_name || ' ' ||v_reg_cursor.last_name);
    FETCH c_empleados INTO v_reg_cursor;
  END LOOP;
  CLOSE c_empleados;
END EMPLEADOS;













```


EXCEPTION
   WHEN ZERO_DIVIDE THEN  -- handles 'division by zero' error
      INSERT INTO stats (symbol, ratio) VALUES ('XYZ', NULL);
      COMMIT;




































## Sobre MongoDB

### Ejercicio 6

> Crea una colección en MongoDB con varios documentos que contengan como mínimo nombre y precio de un producto.





### Ejercicio 7

> Escribe una consulta que muestre aquellos documentos correspondientes a productos cuyo precio sea superior a 20.
