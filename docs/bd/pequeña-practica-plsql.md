# Pequeña práctica de PL/SQL

## Ejercicio 1

> Hacer un procedimiento que muestre el nombre y el salario del empleado cuyo código es 7782

```sql
CREATE OR REPLACE PROCEDURE mostrar_emp_7782
IS
  v_nombre emp.ename%TYPE;
  v_salario emp.sal%TYPE;
BEGIN
  SELECT ename, sal
  INTO v_nombre, v_salario
  FROM emp
  WHERE empno = 7782;

  dbms_output.put_line('El nombre del empleado con codigo 7782 es ' || v_nombre || ' y su salario es ' || v_salario);
END;
/
```

```sql
EXEC mostrar_emp_7782;
```

```shell
El nombre del empleado con codigo 7782 es CLARK y su salario es 2450

PL/SQL procedure successfully completed.
```

## Ejercicio 2

> Hacer un procedimiento que reciba como parámetro un código de empleado y devuelva su nombre

```sql

```



## Ejercicio 3

> Hacer un procedimiento que devuelva los nombres de los tres empleados más antiguos



## Ejercicio 4

> Hacer un procedimiento que reciba el nombre de un tablespace y muestre los nombres de los usuarios que lo tienen como tablespace por defecto (Vista DBA_USERS)




## Ejercicio 5

> Modificar el procedimiento anterior para que haga lo mismo pero devolviendo el número de usuarios que tienen ese tablespace como tablespace por defecto. Nota: Hay que convertir el procedimiento en función






![]()

```shell

```









