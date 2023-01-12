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
```

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
```

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
```

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
```












### Comprobaciones

```sql
EXEC imprimir_boletines(1,'1');
EXEC imprimir_boletines(2,'Alcalde Garcia, Elena');
```


![pruebacurso1](https://i.postimg.cc/c1YDPDY8/boletin1curso.png)















Deben gestionarse las siguientes excepciones: Curso Inexistente (en los informes  tipo 1), Alumno inexistente (en los informes tipo 2) (4 puntos) y No existen notas (en ambos tipos)


## Ejercicio 2

2.- Diseña los módulos necesarios para que un mismo alumno no pueda tener notas de asignaturas de cursos distintos. Es decir, el alumno o es de primero o es de segundo. Si ya tiene notas de uno de los dos cursos, no puede tener notas del otro.

Debe funcionar sin dar problema de tablas mutantes en consultas de datos anexados y en consultas de modificación que afecten a múltiples registros.




























