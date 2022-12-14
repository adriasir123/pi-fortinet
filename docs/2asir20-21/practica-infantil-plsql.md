---
title: "Práctica infantil de PL/SQL"
---

# Ejercicio 1

Haz una función llamada DevolverCodDept que reciba el nombre de un departamento y devuelva su código.

```sql
CREATE OR REPLACE FUNCTION DevolverCodDept
    ( nombre_dept IN varchar2 )
    RETURN number

IS  
    v_cod_dept number;  

BEGIN  
    SELECT deptno into v_cod_dept   
        FROM dept 
        WHERE dname = nombre_dept;

RETURN v_cod_dept;

END;
/
```

Llamada a la función:

> Importante `set serveroutput on` de antemano para que la consola muestre los resultados por pantalla)

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE( 'El código es: ' || DevolverCodDept('RESEARCH') );
END;
/
```

> En el parámetro de entrada de la llamada a la función, cambiar el nombre del departamento según el código de departamento que queramos averiguar.

Resultado:

```
El código es: 20

Procedimiento PL/SQL terminado correctamente.
```


























# Ejercicio 2

Realiza un procedimiento llamado HallarNumEmp que recibiendo un nombre de departamento, muestre en pantalla el número de empleados de dicho departamento. Puedes utilizar la función creada en el ejercicio 1.

Si el departamento no tiene empleados deberá mostrar un mensaje informando de ello. Si el departamento no existe se tratará la excepción correspondiente.

# Ejercicio 3

Realiza una función llamada CalcularCosteSalarial que reciba un nombre de departamento y devuelva la suma de los salarios y comisiones de los empleados de dicho departamento. Trata las excepciones que consideres necesarias.

# Ejercicio 4

Realiza un procedimiento MostrarCostesSalariales que muestre los nombres de todos los departamentos y el coste salarial de cada uno de ellos. Puedes usar la función del ejercicio 3.

# Ejercicio 5

Realiza un procedimiento MostrarAbreviaturas que muestre las tres primeras letras del nombre de cada empleado.

# Ejercicio 6

Realiza un procedimiento MostrarMasAntiguos que muestre el nombre del empleado más antiguo de cada departamento junto con el nombre del departamento. Trata las excepciones que consideres necesarias.

# Ejercicio 7

Realiza un procedimiento MostrarJefes que reciba el nombre de un departamento y muestre los nombres de los empleados de ese departamento que son jefes de otros empleados.Trata las excepciones que consideres necesarias.

# Ejercicio 8

Realiza un procedimiento MostrarMejoresVendedores que muestre los nombres de los dos vendedores con más comisiones. Trata las excepciones que consideres necesarias.

# Ejercicio 9

Realiza un procedimiento MostrarsodaelpmE que reciba el nombre de un departamento al revés y muestre los nombres de los empleados de ese departamento. Trata las excepciones que consideres necesarias.

# Ejercicio 10

Realiza un procedimiento RecortarSueldos que recorte el sueldo un 20% a los empleados cuyo nombre empiece por la letra que recibe como parámetro.Trata las excepciones que consideres necesarias.

# Ejercicio 11

Realiza un procedimiento BorrarBecarios que borre a los dos empleados más nuevos de cada departamento. Trata las excepciones que consideres necesarias.


