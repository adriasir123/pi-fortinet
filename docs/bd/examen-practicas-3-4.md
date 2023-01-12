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
    dbms_output.put_line('Curso: ' || p_cursoalumno);
    dbms_output.put_line('Fecha y hora de impresion: ' || to_char(sysdate, 'DD-MON-YYYY HH24:MI'));
    dbms_output.put_line(chr(9));
    FOR i IN c_notas_por_curso LOOP
        dbms_output.put_line(chr(9) || 'Alumno: ' || i.apenom);
        dbms_output.put_line(chr(9) || 'Direccion: ' || i.direc || ' (' || i.pobla || ')');
        dbms_output.put_line(chr(9) || 'Asignatura: ' || i.nombre);
        dbms_output.put_line(chr(9) || 'Nota: ' || i.nota);
        dbms_output.put_line(chr(9));
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
    SELECT direc INTO vv_direc
    FROM alumnos
    WHERE apenom = p_cursoalumno;
    SELECT pobla INTO vv_pobla
    FROM alumnos
    WHERE apenom = p_cursoalumno;
    dbms_output.put_line('Alumno: ' || p_cursoalumno);
    dbms_output.put_line('Direccion: ' || vv_direc || ' (' || vv_pobla || ')');
    dbms_output.put_line('Fecha y hora de impresion: ' || to_char(sysdate, 'DD-MON-YYYY HH24:MI'));
    dbms_output.put_line(chr(9));
    FOR i IN c_notas_por_alumno LOOP
        dbms_output.put_line(chr(9) || 'Asignatura: ' || i.nombre);
        dbms_output.put_line(chr(9) || 'Nota: ' || i.nota);
        dbms_output.put_line(chr(9));
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




























