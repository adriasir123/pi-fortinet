# Curso de fundamentos de PL/SQL

## Introducción

<https://livesql.oracle.com/apex/f?p=590:1000>

## Fundamentos de PL/SQL

### Práctica: Utilizando funciones SQL en PL/SQL

NVL con variable nula:

```sql
DECLARE
  vv_micadena VARCHAR2(20 CHAR);
BEGIN
  vv_micadena := NVL(vv_micadena, 'Cadena vacia');
  DBMS_OUTPUT.PUT_LINE('El valor de la variable vv_micadena es ' || vv_micadena);
END;
/
```

```sql
El valor de la variable vv_micadena es Cadena vacia

PL/SQL procedure successfully completed.
```

NVL con variable con valor:

```sql
DECLARE
  vv_micadena VARCHAR2(20 CHAR) := 'Un valor';
BEGIN
  vv_micadena := NVL(vv_micadena, 'Cadena vacia');
  DBMS_OUTPUT.PUT_LINE('El valor de la variable vv_micadena es ' || vv_micadena);
END;
/
```

```sql
El valor de la variable vv_micadena es Un valor

PL/SQL procedure successfully completed.
```

Como la variable no era nula, la función NVL no ha cambiado el resultado.

LENGTH:

```sql
DECLARE
  vv_micadena VARCHAR2(20 CHAR) := 'Un valor';
BEGIN
  DBMS_OUTPUT.PUT_LINE('La longitud de la variable vv_micadena es ' || LENGTH(vv_micadena));
END;
/
```

```sql
La longitud de la variable vv_micadena es 8

PL/SQL procedure successfully completed.
```

ROUND:

```sql
DECLARE
  vn_mi_numero NUMBER(3, 2) := 7.77;
BEGIN
  vn_mi_numero := ROUND(vn_mi_numero);
  DBMS_OUTPUT.PUT_LINE('El valor de vn_mi_numero es ' || vn_mi_numero);
END;
/
```

```sql
El valor de vn_mi_numero es 8

PL/SQL procedure successfully completed.
```

Hasta 1 decimal:

```sql
DECLARE
  vn_mi_numero NUMBER(3, 2) := 7.77;
BEGIN
  vn_mi_numero := ROUND(vn_mi_numero, 1);
  DBMS_OUTPUT.PUT_LINE('El valor de vn_mi_numero es ' || vn_mi_numero);
END;
/
```

```sql
El valor de vn_mi_numero es 7.8

PL/SQL procedure successfully completed.
```

TRUNC:

```sql
DECLARE
  vn_mi_numero NUMBER(3, 2) := 7.77;
BEGIN
  vn_mi_numero := TRUNC(vn_mi_numero);
  DBMS_OUTPUT.PUT_LINE('El valor de vn_mi_numero es ' || vn_mi_numero);
END;
/
```

```sql
El valor de vn_mi_numero es 7

PL/SQL procedure successfully completed.
```

Hasta 1 decimal:

```sql
DECLARE
  vn_mi_numero NUMBER(3, 2) := 7.77;
BEGIN
  vn_mi_numero := TRUNC(vn_mi_numero, 1);
  DBMS_OUTPUT.PUT_LINE('El valor de vn_mi_numero es ' || vn_mi_numero);
END;
/
```

```sql
El valor de vn_mi_numero es 7.7

PL/SQL procedure successfully completed.
```

SUBSTR:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := SUBSTR(vv_mi_cadena, 6);
  DBMS_OUTPUT.PUT_LINE('El valor de vv_mi_cadena es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor de vv_mi_cadena es: es mi cadena

PL/SQL procedure successfully completed.
```

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := SUBSTR(vv_mi_cadena, 6, 2);
  DBMS_OUTPUT.PUT_LINE('El valor de vv_mi_cadena es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor de vv_mi_cadena es: es

PL/SQL procedure successfully completed.
```

INSRT:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
  vn_mi_numero NUMBER(2);
BEGIN
  vn_mi_numero := INSTR(vv_mi_cadena, 'mi');
  DBMS_OUTPUT.PUT_LINE('El valor de vn_mi_numero es: ' || vn_mi_numero);
END;
/
```

```sql
El valor de vn_mi_numero es: 9

PL/SQL procedure successfully completed.
```

Devuelve la posición numérica de la primera aparición de la substring buscada dentro de la string.

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
  vn_mi_numero NUMBER(2);
BEGIN
  vn_mi_numero := INSTR(vv_mi_cadena, 'a', 1, 2);
  DBMS_OUTPUT.PUT_LINE('El valor de vn_mi_numero es: ' || vn_mi_numero);
END;
/
```

```sql
El valor de vn_mi_numero es: 13

PL/SQL procedure successfully completed.
```

Devuelve la posición de la segunda ocurrencia de la letra a partir de la ocurrencia 1.

UPPER:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := UPPER(vv_mi_cadena);
  DBMS_OUTPUT.PUT_LINE('El valor de vv_mi_cadena es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor de vv_mi_cadena es: ESTA ES MI CADENA

PL/SQL procedure successfully completed.
```

LOWER:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := LOWER(vv_mi_cadena);
  DBMS_OUTPUT.PUT_LINE('El valor de vv_mi_cadena es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor de vv_mi_cadena es: esta es mi cadena

PL/SQL procedure successfully completed.
```

REPLACE:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := REPLACE(vv_mi_cadena, 'a', '$');
  DBMS_OUTPUT.PUT_LINE('El valor de vv_mi_cadena es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor de vv_mi_cadena es: Est$ es mi c$den$

PL/SQL procedure successfully completed.
```

LPAD:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := LPAD(vv_mi_cadena, 50, '$');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor es: $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$Esta es mi cadena

PL/SQL procedure successfully completed.
```

RPAD:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := RPAD(vv_mi_cadena, 50, '$');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor es: Esta es mi cadena$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

PL/SQL procedure successfully completed.
```

TRIM:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := RPAD(vv_mi_cadena, 50, '$');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vv_mi_cadena);
  vv_mi_cadena := TRIM('$' FROM vv_mi_cadena);
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor es: Esta es mi cadena$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
El valor es: Esta es mi cadena

PL/SQL procedure successfully completed.
```

RTRIM:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := RPAD(vv_mi_cadena, 50, '$');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vv_mi_cadena);
  vv_mi_cadena := RTRIM(vv_mi_cadena, '$');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor es: Esta es mi cadena$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
El valor es: Esta es mi cadena

PL/SQL procedure successfully completed.
```

LTRIM:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := 'Esta es mi cadena';
BEGIN
  vv_mi_cadena := LPAD(vv_mi_cadena, 50, '$');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vv_mi_cadena);
  vv_mi_cadena := LTRIM(vv_mi_cadena, '$');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vv_mi_cadena);
END;
/
```

```sql
El valor es: $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$Esta es mi cadena
El valor es: Esta es mi cadena

PL/SQL procedure successfully completed.
```

TO_NUMBER:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := '10';
  vn_mi_numero NUMBER(2);
BEGIN
  vn_mi_numero := TO_NUMBER(vv_mi_cadena);
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vn_mi_numero);
END;
/
```

```sql
El valor es: 10

PL/SQL procedure successfully completed.
```

TO_DATE:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := '10/05/2022';
  vd_mi_fecha DATE;
BEGIN
  vd_mi_fecha := TO_DATE(vv_mi_cadena, 'DD/MM/YYYY');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || vd_mi_fecha);
END;
/
```

```sql
El valor es: 10-MAY-22

PL/SQL procedure successfully completed.
```

TO_CHAR:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := '10/05/2022';
  vd_mi_fecha DATE;
BEGIN
  vd_mi_fecha := TO_DATE(vv_mi_cadena, 'DD/MM/YYYY');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || TO_CHAR(vd_mi_fecha, 'DD/MM/YYYY'));
END;
/
```

```sql
El valor es: 10/05/2022

PL/SQL procedure successfully completed.
```

ADD_MONTHS:

```sql
DECLARE
  vv_mi_cadena VARCHAR2(50 CHAR) := '10/05/2022';
  vd_mi_fecha DATE;
BEGIN
  vd_mi_fecha := TO_DATE(vv_mi_cadena, 'DD/MM/YYYY');
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || TO_CHAR(vd_mi_fecha, 'DD/MM/YYYY'));
  vd_mi_fecha := ADD_MONTHS(vd_mi_fecha, 1);
  DBMS_OUTPUT.PUT_LINE('El valor es: ' || TO_CHAR(vd_mi_fecha, 'DD/MM/YYYY'));
END;
/
```

```sql
El valor es: 10/05/2022
El valor es: 10/06/2022

PL/SQL procedure successfully completed.
```

### Bloques anidados

```sql
DECLARE
  vv_aplicacion VARCHAR2(10 CHAR) := 'Mi app';
BEGIN
  BEGIN
    vv_aplicacion := vv_aplicacion || ' 2';
  END;
  DBMS_OUTPUT.PUT_LINE('La variable vv_aplicacion contiene el valor ' || vv_aplicacion);
END;
/
```

```sql
La variable vv_aplicacion contiene el valor Mi app 2

PL/SQL procedure successfully completed.
```

Un bloque anidado puede usar las variables del bloque padre.

A la inversa no funciona:

```sql
DECLARE
  vv_aplicacion VARCHAR2(10 CHAR) := 'Mi app';
BEGIN
  DECLARE
    vv_aplicacion VARCHAR2(10 CHAR);
  BEGIN
    vv_aplicacion := 'Subbloque';
  END;
  DBMS_OUTPUT.PUT_LINE('La variable vv_aplicacion contiene el valor ' || vv_aplicacion);
END;
/
```

```sql
La variable vv_aplicacion contiene el valor Mi app

PL/SQL procedure successfully completed.
```

El bloque padre no puede usar variables del bloque hijo.

### Cursores

Se pueden hacer cursores dinámicos, a los que se le pueden pasar parámetros:

```sql
DECLARE
  CURSOR c_proveedores (p_id NUMBER) IS
    SELECT *
    FROM tbproveedores
    WHERE nidproveedor = p_id;
BEGIN
  FOR r_proveedor IN c_proveedores(1) LOOP
    DBMS_OUTPUT.PUT_LINE('El ID del proveedor es ' || r_proveedor.NIDPROVEEDOR);
  END LOOP;
END;
/
```

Así podríamos tener un mismo código de cursor que variaría en resultados.

### Excepciones

```sql
BEGIN
  INSERT INTO dept VALUES (10, 'ACCOUNTING', 'NEW YORK');
EXCEPTION
  WHEN DUP_VAL_ON_INDEX THEN
    DBMS_OUTPUT.PUT_LINE('El registro tiene un ID que ya existe en el sistema');
END;
/
```

```sql
El registro tiene un ID que ya existe en el sistema

PL/SQL procedure successfully completed.
```

Cuando ocurre una excepción se para la ejecución, y no se sigue ejecutando nada más.  
Si queremos que cuando ocurra un error se trate, y siga la ejecución, podemos usar bloques anidados:

```sql
BEGIN
  BEGIN
    INSERT INTO dept VALUES (10, 'ACCOUNTING', 'NEW YORK');
  EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
      DBMS_OUTPUT.PUT_LINE('El registro tiene un ID que ya existe en el sistema');
  END;
  
  DBMS_OUTPUT.PUT_LINE('Test');
END;
/
```

```sql
El registro tiene un ID que ya existe en el sistema
Test

PL/SQL procedure successfully completed.
```

Excepción personalizada nuestra:

```sql
DECLARE
  ex_mi_excepcion EXCEPTION;
BEGIN
  BEGIN
    INSERT INTO dept VALUES (10, 'ACCOUNTING', 'NEW YORK');
  EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
      RAISE ex_mi_excepcion;
  END;
EXCEPTION
  WHEN ex_mi_excepcion THEN
    DBMS_OUTPUT.PUT_LINE('El registro tiene un ID que ya existe en el sistema');
END;
/
```

```sql
El registro tiene un ID que ya existe en el sistema

PL/SQL procedure successfully completed.
```

## Organización del código

### Funciones

Errores en funciones con excepciones:

```sql
CREATE OR REPLACE FUNCTION fn_alta_proveedor (po_v_error VARCHAR2)
  RETURN NUMBER
DECLARE
  ex_mi_excepcion EXCEPTION;
BEGIN
  BEGIN
    INSERT INTO dept VALUES (10, 'ACCOUNTING', 'NEW YORK');
  EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
      RAISE ex_mi_excepcion;
  END;

  RETURN TRUE;
EXCEPTION
  WHEN ex_mi_excepcion THEN
    po_v_error := 'error';
    RETURN FALSE;
END;
/
```

Para llamar a esa función:

```sql
fn_alta_proveedor(vn_IDProveedor, 'Martillos la forja 3', vd_Fecha, vv_ErrorAlta);
```

El error es un parámetro de salida normal.

Control de funciones en el padre que devuelve booleano:

```sql
DECLARE
  vb_AltaOK BOOLEAN;
  vv_ErrorAlta VARCHAR2(2000 CHAR);
BEGIN
  vb_AltaOK := fn_alta_proveedor(vn_IDProveedor, 'Martillos El Hierro', vd_Fecha, vv_ErrorAlta);

  IF vb_AltaOK THEN
    DBMS_OUTPUT.PUT_LINE('El alta se ha realizado correctamente');
  ELSE
    DBMS_OUTPUT.PUT_LINE('El alta no se ha realizado correctamente. Error: ' || vv_ErrorAlta);
  END IF;
END;
/
```

### Práctica: paquetes

Especificación del paquete:

```sql
CREATE OR REPLACE PACKAGE pk_empleados IS
  FUNCTION fn_nombre_empleado (pi_n_idEmpleado emp.empno%TYPE)
    RETURN emp.ename%TYPE;
  PROCEDURE pr_invoca_alta_e (pi_n_idEmpleado emp.empno%TYPE);
END pk_empleados;
/
```

Cuerpo del paquete:

```sql
CREATE OR REPLACE PACKAGE BODY pk_empleados IS

  FUNCTION fn_nombre_empleado (pi_n_idEmpleado emp.empno%TYPE)
    RETURN emp.ename%TYPE
  IS
    vv_nomEmpleado emp.ename%TYPE;
  BEGIN
    SELECT ename
    INTO vv_nomEmpleado
    FROM emp
    WHERE empno = pi_n_idEmpleado;

    RETURN vv_nomEmpleado;
  END fn_nombre_empleado;

  PROCEDURE pr_invoca_alta_e (pi_n_idEmpleado emp.empno%TYPE)
  IS
    vv_nomEmpleado emp.ename%TYPE;
  BEGIN
    vv_nomEmpleado := fn_nombre_empleado(pi_n_idEmpleado => pi_n_idEmpleado);
    DBMS_OUTPUT.PUT_LINE('El nombre del empleado es ' || vv_nomEmpleado);
  END pr_invoca_alta_e;

END pk_empleados;
/
```

Llamo al procedimiento de entrada al paquete:

```sql
BEGIN
  pk_empleados.pr_invoca_alta_e(7876);
END;
/
```

## Otros elementos PL/SQL

### Práctica: Triggers

Muestro el estado actual de `dept`:

```sql
SELECT * FROM dept;
```

![dept](https://i.imgur.com/JNwjkqp.png)

Creo su tabla de log:

```sql
CREATE TABLE LOGDEPT (
  DEPTNO_OLD NUMBER(2),
  DEPTNO_NEW NUMBER(2),
  DNAME_OLD VARCHAR2(14),
  DNAME_NEW VARCHAR2(14),
  LOC_OLD VARCHAR2(13),
  LOC_NEW VARCHAR2(13),
  VACCION VARCHAR2(20)
);
```

Creo el trigger que insertará en la tabla de log por cada INSERT, UPDATE o DELETE en `dept`:

```sql
CREATE OR REPLACE TRIGGER tr_log_departamentos
  AFTER INSERT OR UPDATE OR DELETE ON dept
  FOR EACH ROW
DECLARE
  vv_accion VARCHAR2(20 CHAR);
BEGIN

  CASE
    WHEN INSERTING THEN vv_accion := 'INSERT';
    WHEN UPDATING THEN vv_accion := 'UPDATE';
    WHEN DELETING THEN vv_accion := 'DELETE';
  END CASE;

  INSERT INTO LOGDEPT VALUES (:OLD.deptno, :NEW.deptno, :OLD.dname, :NEW.dname, :OLD.loc, :NEW.loc, vv_accion);

END tr_log_departamentos;
/
```

Para probar el trigger, hago las siguientes operaciones sobre `dept`:

```sql
SQL> INSERT INTO dept VALUES (50, 'ENGINEERING', 'VIRGINIA');

1 row created.

Commit complete.

SQL> UPDATE dept SET dname = 'ENGINEERING 2' WHERE deptno = 50;

1 row updated.

Commit complete.

SQL> DELETE FROM dept WHERE deptno = 50;

1 row deleted.

Commit complete.
```

Muestro el resultado del log:

```sql
SELECT * FROM LOGDEPT;
```

![logdept](https://i.imgur.com/9oitrO4.png)

### Tipos y registros

Registro con TYPE:

```sql
DECLARE
  TYPE t_registro IS RECORD (
    v_producto VARCHAR2(20),
    n_unidades NUMBER(3),
    d_fechaAlta DATE
  );
  vtr_producto t_registro;
BEGIN
  vtr_producto.v_producto := 'Martillo';
  vtr_producto.n_unidades := 10;
  vtr_producto.d_fechaAlta := SYSDATE;

  DBMS_OUTPUT.PUT_LINE('El producto es ' || vtr_producto.v_producto);
  DBMS_OUTPUT.PUT_LINE('Unidades: ' || vtr_producto.n_unidades);
  DBMS_OUTPUT.PUT_LINE('La fecha de alta es ' || TO_CHAR(vtr_producto.d_fechaAlta, 'DD/MM/YYYY'));
END;
/
```

Otra manera de crear un registro:

```sql
DECLARE
  vr_empleados emp%ROWTYPE;
BEGIN
  SELECT *
  INTO vr_empleados
  FROM emp
  WHERE empno = 7876;

  DBMS_OUTPUT.PUT_LINE('El ID del empleado es ' || vr_empleados.empno);
  DBMS_OUTPUT.PUT_LINE('El nombre del empleado es ' || vr_empleados.ename);
  DBMS_OUTPUT.PUT_LINE('La fecha de contratacion es ' || vr_empleados.hiredate);
END;
/
```

### Práctica: Colecciones

Tipo VARRAY:

```sql
DECLARE
  -- Creamos el tipo
  TYPE ttab_ty_nombre IS VARRAY(5) OF VARCHAR2(20 CHAR);
  -- Declaramos e inicializamos una variable del tipo creado
  vttab_ty_nombre ttab_ty_nombre := ttab_ty_nombre();

BEGIN
  
  DBMS_OUTPUT.PUT_LINE('El número máximo de registros en el vector es: ' || vttab_ty_nombre.LIMIT);
  DBMS_OUTPUT.PUT_LINE('El número de registros en el vector es: ' || vttab_ty_nombre.COUNT);
  
  FOR i IN 1 .. vttab_ty_nombre.LIMIT LOOP
    vttab_ty_nombre.EXTEND;
    vttab_ty_nombre(i) := 'Valor ' || TO_CHAR(i);
  END LOOP;

  IF vttab_ty_nombre.COUNT > 0 THEN
    FOR i IN vttab_ty_nombre.FIRST .. vttab_ty_nombre.LAST LOOP
      DBMS_OUTPUT.PUT_LINE('El ID del registro es: ' || vttab_ty_nombre(i));
    END LOOP;
  END IF;

  DBMS_OUTPUT.PUT_LINE('El número de registros en el vector es: ' || vttab_ty_nombre.COUNT);

END;
/
```

```sql
El número máximo de registros en el vector es: 5
El número de registros en el vector es: 0
El ID del registro es: Valor 1
El ID del registro es: Valor 2
El ID del registro es: Valor 3
El ID del registro es: Valor 4
El ID del registro es: Valor 5
El número de registros en el vector es: 5

PL/SQL procedure successfully completed.
```

Otra forma de VARRAY si conocemos los valores:

```sql
DECLARE
  -- Creamos el tipo
  TYPE ttab_ty_nombre IS VARRAY(5) OF VARCHAR2(20 CHAR);
  -- Declaramos e inicializamos una variable del tipo creado
  vttab_ty_nombre ttab_ty_nombre := ttab_ty_nombre(5, 2, 4, 6, 7);

BEGIN
  
  DBMS_OUTPUT.PUT_LINE('El número máximo de registros en el vector es: ' || vttab_ty_nombre.LIMIT);
  DBMS_OUTPUT.PUT_LINE('El número de registros en el vector es: ' || vttab_ty_nombre.COUNT);

  IF vttab_ty_nombre.COUNT > 0 THEN
    FOR i IN vttab_ty_nombre.FIRST .. vttab_ty_nombre.LAST LOOP
      DBMS_OUTPUT.PUT_LINE('El ID del registro es: ' || vttab_ty_nombre(i));
    END LOOP;
  END IF;

  DBMS_OUTPUT.PUT_LINE('El número de registros en el vector es: ' || vttab_ty_nombre.COUNT);

END;
/
```

```sql
El número máximo de registros en el vector es: 5
El número de registros en el vector es: 5
El ID del registro es: 5
El ID del registro es: 2
El ID del registro es: 4
El ID del registro es: 6
El ID del registro es: 7
El número de registros en el vector es: 5

PL/SQL procedure successfully completed.
```

Tabla anidada:

```sql
DECLARE
  -- Creamos el tipo
  TYPE ttab_ty_nombre IS TABLE OF VARCHAR2(20 CHAR);
  -- Declaramos e inicializamos una variable del tipo creado
  vttab_ty_nombre ttab_ty_nombre := ttab_ty_nombre(NULL, NULL, NULL);

BEGIN

  DBMS_OUTPUT.PUT_LINE('El número de registros en el vector es: ' || vttab_ty_nombre.COUNT);

  FOR i IN vttab_ty_nombre.FIRST .. vttab_ty_nombre.LAST LOOP
    vttab_ty_nombre(i) := 'Prueba' || TO_CHAR(i);
  END LOOP;

  IF vttab_ty_nombre.COUNT > 0 THEN
    FOR i IN vttab_ty_nombre.FIRST .. vttab_ty_nombre.LAST LOOP
      DBMS_OUTPUT.PUT_LINE('El ID del registro es: ' || vttab_ty_nombre(i));
    END LOOP;
  END IF;

END;
/
```

```sql
El número de registros en el vector es: 3
El ID del registro es: Prueba1
El ID del registro es: Prueba2
El ID del registro es: Prueba3

PL/SQL procedure successfully completed.
```

Vector asociativo:

```sql
DECLARE
  -- Creamos el tipo
  TYPE ttab_ty_nombre IS TABLE OF VARCHAR2(20 CHAR) INDEX BY VARCHAR2(2);
  -- Declaramos una variable del tipo creado
  vttab_ty_nombre ttab_ty_nombre;

  v_indice VARCHAR2(2);

BEGIN

  DBMS_OUTPUT.PUT_LINE('El número de registros en el vector es: ' || vttab_ty_nombre.COUNT);

  vttab_ty_nombre('ES') := 'España';
  vttab_ty_nombre('AR') := 'Argentina';
  vttab_ty_nombre('ME') := 'Mexico';
  vttab_ty_nombre('CO') := 'Colombia';

  v_indice := vttab_ty_nombre.FIRST;
  DBMS_OUTPUT.PUT_LINE('El indice del registro es: ' || v_indice);

  WHILE v_indice IS NOT NULL LOOP
    DBMS_OUTPUT.PUT_LINE('El ID es: ' || v_indice || ' y el valor es: ' || vttab_ty_nombre(v_indice));
    v_indice := vttab_ty_nombre.NEXT(v_indice);
  END LOOP;
  
  DBMS_OUTPUT.PUT_LINE('Acceso directo: El ID es ES y su valor es: ' || vttab_ty_nombre('ES'));
  DBMS_OUTPUT.PUT_LINE('El número de registros en el vector es: ' || vttab_ty_nombre.COUNT);

END;
/
```

```sql
El número de registros en el vector es: 0
El indice del registro es: AR
El ID es: AR y el valor es: Argentina
El ID es: CO y el valor es: Colombia
El ID es: ES y el valor es: España
El ID es: ME y el valor es: Mexico
Acceso directo: El ID es ES y su valor es: España
El número de registros en el vector es: 4

PL/SQL procedure successfully completed.
```
