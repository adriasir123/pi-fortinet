# Examen prácticas 3 y 4

## Pasos previos

Me conecto como sys y creo el usuario a usar:

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

Realiza un procedimiento llamado imprimir_boletines que reciba dos parámetros.

El primer parámetro será numérico y podrá tener los siguientes valores:

- Un 1 si se desea imprimir todos los boletines de un curso concreto

- Un 2 si queremos imprimir el boletín de un alumno concreto

El segundo parámetro será el curso del que se desean imprimir los boletines (en el primer caso) o el nombre del alumno del que se desea imprimir el boletín (en el segundo caso).



```sql
CREATE OR REPLACE PROCEDURE boletintipo1 (
    p_curso asignaturas.curso%type
) IS
  CURSOR c_boletintipo1 IS
    SELECT al.dni, al.apenom, al.direc, al.pobla, al.telef, n.cod, n.nota, a.nombre
    FROM alumnos al
    INNER JOIN notas n
      ON al.dni = n.dni
    INNER JOIN asignaturas a
      ON n.cod = a.cod
    WHERE a.curso = p_curso;
BEGIN
    dbms_output.put_line('Curso: ' || p_curso);
    dbms_output.put_line(chr(9));
    FOR i IN c_boletintipo1 LOOP
      dbms_output.put_line(CHR(9) || 'Nombre y apellidos: ' || i.apenom);
      dbms_output.put_line(CHR(9) || 'Direccion: ' || i.direc);
      DBMS_OUTPUT.PUT_LINE(CHR(9) || 'Fecha y hora de impresion: ' || TO_CHAR(SYSDATE, 'DD-MON-YYYY HH24:MI:SS'));
      dbms_output.put_line(CHR(9) || 'Asignatura: ' || i.nombre);
      dbms_output.put_line(CHR(9) || 'Nota: ' || i.nota);
      dbms_output.put_line(chr(9));
    END LOOP;
END;
/
```







En cada boletín debe aparecer:

- el nombre y apellidos del alumno
- su dirección completa
- la fecha y hora de impresión
- y una lista con los nombres de las asignaturas y la nota conseguida en cada una de ellas por el alumno





```sql
CREATE OR REPLACE PROCEDURE imprimir_boletines(
    p_tipo NUMBER,
    p_curso asignaturas.curso%type,
    p_alumno alumnos.apenom%type
) IS
BEGIN
    IF p_tipo = 1 THEN
        boletintipo1(p_curso);
    ELSIF p_tipo = 2 THEN
        --boletintipo2(p_alumno);
        dbms_output.put_line('test');
    END IF;
END;
/
```






```sql
EXEC imprimir_boletines(1,1,'');
```


![pruebacurso1](https://i.postimg.cc/c1YDPDY8/boletin1curso.png)















Deben gestionarse las siguientes excepciones: Curso Inexistente (en los informes  tipo 1), Alumno inexistente (en los informes tipo 2) (4 puntos) y No existen notas (en ambos tipos)


## Ejercicio 2

2.- Diseña los módulos necesarios para que un mismo alumno no pueda tener notas de asignaturas de cursos distintos. Es decir, el alumno o es de primero o es de segundo. Si ya tiene notas de uno de los dos cursos, no puede tener notas del otro.

Debe funcionar sin dar problema de tablas mutantes en consultas de datos anexados y en consultas de modificación que afecten a múltiples registros. (5 puntos).




























