# Examen prácticas 3 y 4

## Pasos previos

Me conecto como sys y creo el usuario:

```sql
sqlplus / as sysdba
CREATE USER examenbd34 IDENTIFIED BY 1234;
GRANT ALL PRIVILEGES TO examenbd34;
exit
```

Descargo el script:

```shell
wget https://gist.githubusercontent.com/adriasir123/7115290af0d999c35f9315b788adb3d9/raw/359c4853508068d731302540323d569ac05654da/script-examen-bd-3-4.sql
```

Me conecto como dicho usuario:

```shell
sqlplus examenbd34/1234
```

Ejecuto el script:

```sql
@script-examen-bd-3-4.sql
```

El esquema relacional es el siguiente...

| ALUMNOS   |                     |
|-----------|---------------------|
| DNI       | VARCHAR2, tamaño 10 |
| Apenom    | VARCHAR2, tamaño 36 |
| Direc     | VARCHAR2, tamaño 30 |
| Población | VARCHAR2, tamaño 15 |
| Telefono  | VARCHAR2, tamaño 10 |

| ASIGNATURAS |                     |
|-------------|---------------------|
| Cod         | NUMBER              |
| Nombre      | VARCHAR2, tamaño 26 |
| Curso       | NUMBER              |

| NOTAS |                     |
|-------|---------------------|
| DNI   | VARCHAR2, tamaño 10 |
| Cod   | NUMBER              |
| Nota  | NUMBER              |

## Ejercicio 1

### Enunciado

Realiza un procedimiento llamado `imprimir_boletines` que reciba dos parámetros.

El primer parámetro será numérico y podrá tener los siguientes valores:

- Un 1 si se desea imprimir todos los boletines de un curso concreto

- Un 2 si queremos imprimir el boletín de un alumno concreto

El segundo parámetro será el curso del que se desean imprimir los boletines (en el primer caso) o el nombre del alumno del que se desea imprimir el boletín (en el segundo caso).

En cada boletín debe aparecer el nombre y apellidos del alumno, su dirección completa, la fecha y hora de impresión y una lista con los nombres de las asignaturas y la nota conseguida en cada una de ellas por el alumno.

Deben gestionarse las siguientes excepciones: Curso Inexistente (en los informes  tipo 1), Alumno inexistente (en los informes tipo 2) y No existen notas (en ambos tipos)

### Código

```sql
CREATE OR REPLACE PROCEDURE comprobar_excepciones_tipo1 (
    p_cursoalumno VARCHAR2
) IS
  v_curso_recs NUMBER;
  v_notas_curso NUMBER;
  e_curso_inexistente exception;
  e_curso_sin_notas exception;
BEGIN
  SELECT COUNT(*) INTO v_curso_recs
  FROM asignaturas
  WHERE curso = to_number(p_cursoalumno, '9');
  IF v_curso_recs = 0 THEN
    raise e_curso_inexistente;
  END IF;
  SELECT COUNT(*) INTO v_notas_curso
  FROM notas n
      INNER JOIN asignaturas a
      ON n.cod = a.cod
  WHERE a.curso = to_number(p_cursoalumno, '9');
  IF v_notas_curso = 0 THEN
    raise e_curso_sin_notas;
  END IF;
EXCEPTION
  WHEN e_curso_inexistente THEN
    RAISE_APPLICATION_ERROR(-20001, 'No existe el curso ' || p_cursoalumno);
  WHEN e_curso_sin_notas THEN
    RAISE_APPLICATION_ERROR(-20002, 'El curso ' || p_cursoalumno || ' existe, pero no tiene notas');
END;
/
-- (1)!
```

1. Este procedimiento maneja las excepciones de `boletintipo1`:

    - Curso inexistente
    - Curso sin notas

```sql
CREATE OR REPLACE PROCEDURE comprobar_excepciones_tipo2 (
    p_cursoalumno VARCHAR2
) IS
  v_alumno_recs NUMBER;
  v_notas_alumno NUMBER;
  e_alumno_inexistente exception;
  e_alumno_sin_notas exception;
BEGIN
  SELECT COUNT(*) INTO v_alumno_recs
  FROM alumnos
  WHERE apenom = p_cursoalumno;
  IF v_alumno_recs = 0 THEN
    raise e_alumno_inexistente;
  END IF;
  SELECT COUNT(*) INTO v_notas_alumno
  FROM notas n
      INNER JOIN alumnos a
      ON n.dni = a.dni
  WHERE a.apenom = p_cursoalumno;
  IF v_notas_alumno = 0 THEN
    raise e_alumno_sin_notas;
  END IF;
EXCEPTION
  WHEN e_alumno_inexistente THEN
    RAISE_APPLICATION_ERROR(-20001, 'No existe el alumno ' || p_cursoalumno);
  WHEN e_alumno_sin_notas THEN
    RAISE_APPLICATION_ERROR(-20002, 'El alumno ' || p_cursoalumno || ' existe, pero no tiene notas');
END;
/
-- (1)!
```

1. Este procedimiento maneja las excepciones de `boletintipo2`:

    - Alumno inexistente
    - Alumno sin notas

```sql
CREATE OR REPLACE PROCEDURE boletintipo1 (
    p_cursoalumno VARCHAR2
) IS
    CURSOR c_notas_por_curso IS
        SELECT al.apenom, al.direc, al.pobla, n.nota, a.nombre
        FROM alumnos al
            INNER JOIN notas n
            ON al.dni = n.dni
            INNER JOIN asignaturas a
            ON n.cod = a.cod
        WHERE a.curso = to_number(p_cursoalumno, '9');
BEGIN
    comprobar_excepciones_tipo1(p_cursoalumno);
    DBMS_OUTPUT.PUT_LINE('Curso: ' || p_cursoalumno);
    DBMS_OUTPUT.PUT_LINE('Fecha y hora de impresion: ' || to_char(sysdate, 'DD-MON-YYYY HH24:MI'));
    DBMS_OUTPUT.PUT_LINE(chr(9));
    FOR i IN c_notas_por_curso LOOP
        DBMS_OUTPUT.PUT_LINE(chr(9) || 'Alumno: ' || i.apenom);
        DBMS_OUTPUT.PUT_LINE(chr(9) || 'Direccion: ' || i.direc || ' (' || i.pobla || ')');
        DBMS_OUTPUT.PUT_LINE(chr(9) || 'Asignatura: ' || i.nombre);
        DBMS_OUTPUT.PUT_LINE(chr(9) || 'Nota: ' || i.nota);
        DBMS_OUTPUT.PUT_LINE(chr(9));
    END LOOP;
END;
/
-- (1)!
```

1. Este procedimiento saca un informe de notas según el curso que recibe como parámetro

```sql
CREATE OR REPLACE PROCEDURE boletintipo2 (
    p_cursoalumno VARCHAR2
) IS
    vv_direc alumnos.direc%type;
    vv_pobla alumnos.pobla%type;
    CURSOR c_notas_por_alumno IS
        SELECT n.nota, a.nombre
        FROM alumnos al
            INNER JOIN notas n
            ON al.dni = n.dni
            INNER JOIN asignaturas a
            ON n.cod = a.cod
        WHERE al.apenom = p_cursoalumno;
BEGIN
    comprobar_excepciones_tipo2(p_cursoalumno);
    SELECT direc INTO vv_direc
    FROM alumnos
    WHERE apenom = p_cursoalumno;
    SELECT pobla INTO vv_pobla
    FROM alumnos
    WHERE apenom = p_cursoalumno;
    DBMS_OUTPUT.PUT_LINE('Alumno: ' || p_cursoalumno);
    DBMS_OUTPUT.PUT_LINE('Direccion: ' || vv_direc || ' (' || vv_pobla || ')');
    DBMS_OUTPUT.PUT_LINE('Fecha y hora de impresion: ' || to_char(sysdate, 'DD-MON-YYYY HH24:MI'));
    DBMS_OUTPUT.PUT_LINE(chr(9));
    FOR i IN c_notas_por_alumno LOOP
        DBMS_OUTPUT.PUT_LINE(chr(9) || 'Asignatura: ' || i.nombre);
        DBMS_OUTPUT.PUT_LINE(chr(9) || 'Nota: ' || i.nota);
        DBMS_OUTPUT.PUT_LINE(chr(9));
    END LOOP;
END;
/
-- (1)!
```

1. Este procedimiento saca un informe de notas según el alumno que recibe como parámetro

```sql
CREATE OR REPLACE PROCEDURE imprimir_boletines(
    p_tipo NUMBER,
    p_cursoalumno VARCHAR2
) IS
BEGIN
    IF p_tipo = 1 THEN
        boletintipo1(p_cursoalumno);
    ELSE
        boletintipo2(p_cursoalumno);
    END IF;
END;
/
-- (1)!
```

1. Este es el procedimiento principal, y desde aquí se llama a cada boletín según el tipo que recibe por parámetro. Tiene un segundo parámetro, que recibirá un curso o un alumno, según el tipo de boletín que se quiera imprimir

### Comprobaciones

Compilaciones:

| ![comp1](https://i.imgur.com/SF4HMMY.png) |
|:--:|
| *comprobar_excepciones_tipo1* |

| ![comp2](https://i.imgur.com/EpIFMgs.png) |
|:--:|
| *comprobar_excepciones_tipo2* |

| ![comp3](https://i.imgur.com/vqF6tOB.png) |
|:--:|
| *boletintipo1* |

| ![comp4](https://i.imgur.com/jg6B4Y8.png) |
|:--:|
| *boletintipo2* |

| ![comp5](https://i.imgur.com/HTi9X71.png) |
|:--:|
| *imprimir_boletines* |

Informe sin errores de tipo 1:

```sql
EXEC imprimir_boletines(1,'1');
```

![boletin1normal](https://i.postimg.cc/3JyP8VRr/boletin1curso.png)

Informe sin errores de tipo 2:

```sql
EXEC imprimir_boletines(2,'Alcalde Garcia, Elena');
```

![boletin2normal](https://i.postimg.cc/Wp7MD1dX/boletin2elena.png)

Informe de tipo 1 cuando el curso no existe:

```sql
EXEC imprimir_boletines(1,'9');
```

![boletin1cursonoexiste](https://i.imgur.com/JA2auc5.png)

Informe de tipo 1 cuando el curso existe, pero no tiene notas:

```sql
EXEC imprimir_boletines(1,'3');
```

![boletin1cursoexistesinotas](https://i.imgur.com/17FPWcG.png)

Para que esta prueba haya funcionado, he tenido que añadir una nueva asignatura en el nuevo curso 3:

```sql
INSERT INTO ASIGNATURAS VALUES (8,'Nueva',3);
```

Informe de tipo 2 cuando el alumno no existe:

```sql
EXEC imprimir_boletines(2,'Jim Carrey');
```

![boletin2alumnonoexiste](https://i.imgur.com/IlRf5Di.png)

Informe de tipo 2 cuando el alumno existe, pero no tiene notas:

```sql
EXEC imprimir_boletines(2,'Robin Williams');
```

![boletin2alumnoexistesinotas](https://i.imgur.com/0abdUdE.png)

Para que esta prueba haya funcionado, he tenido que añadir un nuevo alumno:

```sql
INSERT INTO ALUMNOS VALUES ('12344346','Robin Williams', 'C/Adams Boulevard, 33','California','817766545');
```

## Ejercicio 2

Diseña los módulos necesarios para que un mismo alumno no pueda tener notas de asignaturas de cursos distintos. Es decir, el alumno o es de primero o es de segundo. Si ya tiene notas de uno de los dos cursos, no puede tener notas del otro.

Debe funcionar sin dar problema de tablas mutantes en consultas de datos anexados y en consultas de modificación que afecten a múltiples registros



```sql
CREATE OR REPLACE PROCEDURE comprobar_curso_alumno(
    p_dni VARCHAR2,
    p_control_curso OUT NUMBER
) IS
BEGIN
    SELECT AVG(a.curso) INTO p_control_curso
    FROM notas n
        INNER JOIN asignaturas a
        ON n.cod = a.cod
    WHERE n.dni = p_dni;
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE comprobar_curso_asignatura(
    p_cod NUMBER,
    p_control_curso_asignatura OUT NUMBER
) IS
BEGIN
    SELECT curso INTO p_control_curso_asignatura
    FROM asignaturas
    WHERE cod = p_cod;
END;
/
```

```sql
CREATE OR REPLACE TRIGGER monitorizar_notas
    BEFORE INSERT OR UPDATE ON notas
    FOR EACH ROW
DECLARE
    vn_control_curso_alumno NUMBER;
    vn_control_curso_asignatura NUMBER;
BEGIN
    comprobar_curso_alumno(:NEW.dni,vn_control_curso_alumno);
    comprobar_curso_asignatura(:NEW.cod,vn_control_curso_asignatura);

    IF vn_control_curso_alumno != vn_control_curso_asignatura THEN
        RAISE_APPLICATION_ERROR(-20001, 'El alumno con DNI ' || :NEW.dni || ' esta en el curso ' || vn_control_curso_alumno || ' y la asignatura con codigo ' || :NEW.cod || ' es del curso ' || vn_control_curso_asignatura);
    END IF;
END;
/
```



```sql
INSERT INTO NOTAS VALUES('12344345', 1,7);
DELETE FROM NOTAS WHERE dni = '12344345' AND cod = 1 AND nota = 7;
INSERT INTO NOTAS VALUES('12344345',4,7);

UPDATE NOTAS
SET dni = '12344345', cod = 4, nota = 6
WHERE dni = 12344345 AND cod = 1 AND nota = 6;
```




```sql
CREATE OR REPLACE TRIGGER monitorizar_notas_compound    
    FOR UPDATE OR INSERT ON notas    
    COMPOUND TRIGGER

AFTER EACH ROW
```

```sql
```













