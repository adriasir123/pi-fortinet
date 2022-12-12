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
CREATE OR REPLACE PROCEDURE nombre_empleado(p_empno emp.empno%TYPE)
IS
  v_nombre emp.ename%TYPE;
BEGIN
  SELECT ename
  INTO v_nombre
  FROM emp
  WHERE empno = p_empno;
    
  dbms_output.put_line('El nombre del empleado con codigo ' || p_empno || ' es ' || v_nombre);
END;
/
```

```sql
EXEC nombre_empleado(7876);
```

```shell
El nombre del empleado con codigo 7876 es ADAMS

PL/SQL procedure successfully completed.
```

## Ejercicio 3

> Hacer un procedimiento que devuelva los nombres de los tres empleados más antiguos

```sql
CREATE OR REPLACE PROCEDURE empleados_antiguos
IS
  CURSOR c_empleados
  IS
    SELECT ename
    FROM emp
    ORDER BY hiredate ASC
    FETCH FIRST 3 ROWS ONLY;
BEGIN
  dbms_output.put_line('Los tres empleados mas antiguos son:');
  FOR i in c_empleados
  LOOP
    dbms_output.put_line(i.ename);
  END LOOP;
END;
/
```

```sql
EXEC empleados_antiguos;
```

```shell
Los tres empleados mas antiguos son:
SMITH
ALLEN
WARD

PL/SQL procedure successfully completed.
```

## Ejercicio 4

> Hacer un procedimiento que reciba el nombre de un tablespace y muestre los nombres de los usuarios que lo tienen como tablespace por defecto

```sql
CREATE OR REPLACE PROCEDURE tablespace_usuarios(p_tablespace DBA_USERS.DEFAULT_TABLESPACE%TYPE)
IS
  CURSOR c_usuarios
  IS
    SELECT username
    FROM dba_users
    WHERE default_tablespace = p_tablespace;
BEGIN
  dbms_output.put_line('El nombre del tablespace que has escrito es ' || p_tablespace || ' y los usuarios que tienen este tablespace por defecto son:');
  dbms_output.put_line(chr(5));
  FOR i in c_usuarios
  LOOP
    dbms_output.put_line(i.username);
  END LOOP;
END;
/
```

```sql
EXEC tablespace_usuarios('USERS');
```

```shell
El nombre del tablespace que has escrito es USERS y los usuarios que tienen este tablespace por defecto son:

GSMCATUSER
MDDATA
SYSBACKUP
REMOTE_SCHEDULER_AGENT
GSMUSER
SYSRAC
GSMROOTUSER
SI_INFORMTN_SCHEMA
AUDSYS
DIP
ORDPLUGINS
SYSKM
ORDDATA
ORACLE_OCM
SCOTT
SYSDG
ORDSYS

PL/SQL procedure successfully completed.
```

## Ejercicio 5

> Modificar el procedimiento anterior para que haga lo mismo pero devolviendo el número de usuarios que tienen ese tablespace como tablespace por defecto

```sql
CREATE OR REPLACE FUNCTION num_usuarios_tablespace(p_tablespace DBA_USERS.DEFAULT_TABLESPACE%TYPE)
RETURN NUMBER
IS
  v_num_usuarios NUMBER;
BEGIN
  SELECT COUNT(*)
  INTO v_num_usuarios
  FROM dba_users
  WHERE default_tablespace = p_tablespace;
  
  RETURN v_num_usuarios;
END;
/
```

```sql
SELECT num_usuarios_tablespace('USERS') FROM dual;
```

```shell
NUM_USUARIOS_TABLESPACE('USERS')
--------------------------------
		                      17
```

## Ejercicio 6

> Hacer un procedimiento llamado mostrar_usuarios_por_tablespace que muestre por pantalla un listado de los tablespaces existentes con la lista de usuarios de cada uno y el número de los mismos

```sql
CREATE OR REPLACE PROCEDURE proc_listar_usuarios_tablespace(p_tablespace DBA_USERS.DEFAULT_TABLESPACE%TYPE)
IS
  CURSOR c_usuarios
  IS
    SELECT username
    FROM dba_users
    WHERE default_tablespace = p_tablespace;
BEGIN
  FOR i in c_usuarios
  LOOP
    dbms_output.put_line(i.username);
  END LOOP;
END;
/
```

```sql
CREATE OR REPLACE FUNCTION total_usuarios_bd
RETURN NUMBER
IS
  v_total_usuarios NUMBER;
BEGIN
  SELECT COUNT(*)
  INTO v_total_usuarios
  FROM dba_users;
  
  RETURN v_total_usuarios;
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE mostrar_usuarios_por_tablespace
IS
  CURSOR c_tablespaces
  IS
    SELECT tablespace_name
    FROM dba_tablespaces;
BEGIN
  FOR i IN c_tablespaces
  LOOP
    dbms_output.put_line('Tablespace ' || i.tablespace_name || ':');
    dbms_output.put_line(chr(9));
    proc_listar_usuarios_tablespace(i.tablespace_name);
    dbms_output.put_line(chr(9));
    dbms_output.put_line('Total Usuarios Tablespace ' || i.tablespace_name || ': ' || num_usuarios_tablespace(i.tablespace_name));
    dbms_output.put_line(chr(9));
  END LOOP;
  dbms_output.put_line('Total Usuarios BD: ' || total_usuarios_bd());
END;
/
```

```sql
EXEC mostrar_usuarios_por_tablespace;
```

```shell
Tablespace SYSTEM:

SYS
SYSTEM
XS$NULL
OJVMSYS
LBACSYS
OUTLN
SYS$UMF

Total Usuarios Tablespace SYSTEM: 7

Tablespace SYSAUX:

DBSNMP
APPQOSSYS
DBSFWUSER
GGSYS
ANONYMOUS
CTXSYS
DVSYS
DVF
GSMADMIN_INTERNAL
MDSYS
OLAPSYS
XDB
WMSYS

Total Usuarios Tablespace SYSAUX: 13

Tablespace UNDOTBS1:


Total Usuarios Tablespace UNDOTBS1: 0

Tablespace TEMP:


Total Usuarios Tablespace TEMP: 0

Tablespace USERS:

GSMCATUSER
MDDATA
SYSBACKUP
REMOTE_SCHEDULER_AGENT
GSMUSER
SYSRAC
GSMROOTUSER
SI_INFORMTN_SCHEMA
AUDSYS
DIP
ORDPLUGINS
SYSKM
ORDDATA
ORACLE_OCM
SCOTT
SYSDG
ORDSYS

Total Usuarios Tablespace USERS: 17

Total Usuarios BD: 37

PL/SQL procedure successfully completed.
```

## Ejercicio 7

> Hacer un procedimiento llamado mostrar_codigo_fuente que reciba el nombre de otro procedimiento y muestre su código fuente

```sql
CREATE OR REPLACE PROCEDURE mostrar_codigo_fuente(p_procedimiento VARCHAR2)
IS
  CURSOR c_codigo_fuente
  IS
    SELECT text
    FROM dba_source
    WHERE name = p_procedimiento;
BEGIN
  FOR i IN c_codigo_fuente
  LOOP
    dbms_output.put_line(i.text);
  END LOOP;
END;
/
```

```sql
EXEC mostrar_codigo_fuente('TOTAL_USUARIOS_BD');
```

```shell
FUNCTION total_usuarios_bd

RETURN NUMBER

IS

v_total_usuarios NUMBER;

BEGIN

SELECT COUNT(*)

INTO v_total_usuarios

FROM dba_users;



RETURN v_total_usuarios;

END;

PL/SQL procedure successfully completed.
```

## Ejercicio 8

> Hacer un procedimiento llamado mostrar_privilegios_usuario que reciba el nombre de un usuario y muestre sus privilegios de sistema y sus privilegios sobre objetos

```sql
CREATE OR REPLACE PROCEDURE mostrar_privilegios_usuario(p_usuario VARCHAR2)
IS
  CURSOR c_privilegios_sistema
  IS
    SELECT privilege
    FROM dba_sys_privs
    WHERE grantee = p_usuario;
  CURSOR c_privilegios_objetos
  IS
    SELECT privilege, table_name
    FROM dba_tab_privs
    WHERE grantee = p_usuario;
BEGIN
  dbms_output.put_line('El usuario recibido ha sido: ' || p_usuario);
  dbms_output.put_line(chr(9));
  dbms_output.put_line('Sus privilegios de sistema son:');
  FOR i IN c_privilegios_sistema
  LOOP
    dbms_output.put_line(i.privilege);
  END LOOP;
  dbms_output.put_line('Sus privilegios sobre objetos son:');
  FOR i IN c_privilegios_objetos
  LOOP
    dbms_output.put_line(i.privilege || ' sobre ' || i.table_name);
  END LOOP;
END;
/
```

```sql
EXEC mostrar_privilegios_usuario('SYS');
```

[Output](https://gist.github.com/adriasir123/4b68f6910cdcbca08ab7556c14f06289)

## Ejercicio 9

> Realiza un procedimiento PLSQL llamado listar_comisiones que nos muestre por pantalla un listado de las comisiones de los empleados agrupados según la localidad donde está ubicado su departamento.  
> Hay que seguir las siguientes directrices:
>
> - Los nombres de localidades, departamentos y empleados deben aparecer por orden alfabético
> - Si alguno de los departamentos no tiene ningún empleado con comisiones, aparecerá un mensaje informando de ello en lugar de la lista de empleados
> - El procedimiento debe gestionar adecuadamente las siguientes excepciones:
>   - La tabla Empleados está vacía
>   - Alguna comisión es mayor que 10000

```sql
CREATE OR REPLACE PROCEDURE listar_empleados(p_deptno dept.deptno%TYPE)
IS
  CURSOR c_empleados
  IS
    SELECT ename, comm
    FROM emp
    WHERE deptno = p_deptno
    ORDER BY ename ASC;
BEGIN
  FOR i in c_empleados
  LOOP
    dbms_output.put_line(chr(9) || chr(9) || i.ename || ' ..... ' || i.comm);
  END LOOP;
END;
/
```

```sql
CREATE OR REPLACE FUNCTION total_comms_dep(p_deptno dept.deptno%TYPE)
RETURN NUMBER
IS
  v_total_comms NUMBER;
BEGIN
  SELECT NVL( SUM( NVL(comm,0) ),0 )
  INTO v_total_comms
  FROM emp
  WHERE deptno = p_deptno;
  
  RETURN v_total_comms;
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE listar_departamentos(p_localidad dept.loc%TYPE)
IS
  CURSOR c_departamentos
  IS
    SELECT deptno, dname
    FROM dept
    WHERE loc = p_localidad
    ORDER BY dname ASC;
BEGIN
  FOR i in c_departamentos
  LOOP
    dbms_output.put_line(chr(9) || 'Departamento: ' || i.dname);
    dbms_output.put_line(chr(9));
    IF total_comms_dep(i.deptno) = 0 THEN
      dbms_output.put_line(chr(9) || chr(9) || 'No hay ningun empleado con comisiones');
    ELSE
      listar_empleados(i.deptno);
    END IF;
    dbms_output.put_line(chr(9));
    dbms_output.put_line(chr(9) || 'Total Comisiones en el Departamento ' || i.dname || ': ' || total_comms_dep(i.deptno));
  END LOOP;
END;
/
```

```sql
CREATE OR REPLACE FUNCTION total_comms_loc(p_loc dept.loc%TYPE)
RETURN NUMBER
IS
  v_total_comms NUMBER;
BEGIN
  SELECT NVL( SUM( NVL(e.comm,0) ),0 )
  INTO v_total_comms
  FROM emp e
  INNER JOIN dept d
  ON e.deptno = d.deptno
  WHERE d.loc = p_loc;
  
  RETURN v_total_comms;
END;
/
```

```sql
CREATE OR REPLACE FUNCTION total_comms_empresa
RETURN NUMBER
IS
  v_total_comms NUMBER;
BEGIN
  SELECT SUM(comm)
  INTO v_total_comms
  FROM emp;
  
  RETURN v_total_comms;
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE comprobar_excepciones
IS
  v_emp_recs NUMBER;
  v_emp_comms NUMBER;
  e_comm_alta exception;
  e_tabla_vacia_emp exception;
BEGIN
  SELECT COUNT(*) INTO v_emp_recs
  FROM emp;
  IF v_emp_recs = 0 THEN
    raise e_tabla_vacia_emp;
  END IF;
  SELECT COUNT(*) INTO v_emp_comms
  FROM emp
  WHERE comm > 10000;
  IF v_emp_comms > 0 THEN
    raise e_comm_alta;
  END IF;
EXCEPTION
  WHEN e_tabla_vacia_emp THEN
    raise_application_error(-20001, 'La tabla empleados esta vacia');
  WHEN e_comm_alta THEN
    raise_application_error(-20002, 'Alguna comision es mayor que 10000');
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE listar_comisiones
IS
  CURSOR c_localidades
  IS
    SELECT loc
    FROM dept
    ORDER BY loc ASC;
BEGIN
  comprobar_excepciones();
  FOR i IN c_localidades
  LOOP
    dbms_output.put_line('Localidad ' || i.loc);
    dbms_output.put_line(chr(9));
    listar_departamentos(i.loc);
    dbms_output.put_line(chr(9));
    dbms_output.put_line('Total Comisiones en la Localidad ' || i.loc || ': ' || total_comms_loc(i.loc));
    dbms_output.put_line(chr(9));
  END LOOP;
  dbms_output.put_line('Total Comisiones en la Empresa: ' || total_comms_empresa());
END;
/
```

```sql
EXEC listar_comisiones;
```

Output sin excepciones:

```shell
Localidad BOSTON

	Departamento: OPERATIONS

	        No hay ningun empleado con comisiones

	Total Comisiones en el Departamento OPERATIONS: 0

Total Comisiones en la Localidad BOSTON: 0

Localidad CHICAGO

	Departamento: SALES

	        ALLEN ..... 300
	        BLAKE .....
	        JAMES .....
	        MARTIN ..... 1400
	        TURNER ..... 0
	        WARD ..... 500

	Total Comisiones en el Departamento SALES: 2200

Total Comisiones en la Localidad CHICAGO: 2200

Localidad DALLAS

	Departamento: RESEARCH

	        No hay ningun empleado con comisiones

	Total Comisiones en el Departamento RESEARCH: 0

Total Comisiones en la Localidad DALLAS: 0

Localidad NEW YORK

	Departamento: ACCOUNTING

	        No hay ningun empleado con comisiones

	Total Comisiones en el Departamento ACCOUNTING: 0

Total Comisiones en la Localidad NEW YORK: 0

Total Comisiones en la Empresa: 2200

PL/SQL procedure successfully completed.
```

Output con la tabla `emp` vacía:

```shell
BEGIN listar_comisiones; END;

*
ERROR at line 1:
ORA-20001: La tabla empleados esta vacia
ORA-06512: at "SCOTT.COMPROBAR_EXCEPCIONES", line 21
ORA-06512: at "SCOTT.LISTAR_COMISIONES", line 9
ORA-06512: at line 1
```

Output con la tabla `emp` llena, pero alguna comisión mayor que 10000:

```shell
BEGIN listar_comisiones; END;

*
ERROR at line 1:
ORA-20002: Alguna comision es mayor que 10000
ORA-06512: at "SCOTT.COMPROBAR_EXCEPCIONES", line 23
ORA-06512: at "SCOTT.LISTAR_COMISIONES", line 9
ORA-06512: at line 1
```

## Ejercicio 10

> Realiza un procedimiento que reciba el nombre de una tabla y muestre los nombres de las restricciones que tiene, a qué columna afectan, y en qué consisten exactamente

```sql
CREATE OR REPLACE FUNCTION tipo_constraint(p_tabla VARCHAR2, p_constraint VARCHAR2)
RETURN VARCHAR2
IS
  v_tipo_constraint VARCHAR2(20);
BEGIN
  SELECT constraint_type
  INTO v_tipo_constraint
  FROM dba_constraints
  WHERE table_name = p_tabla AND constraint_name = p_constraint;
  RETURN v_tipo_constraint;
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE mostrar_restricciones_tabla(p_tabla VARCHAR2)
IS
  CURSOR c_restricciones
  IS
    SELECT constraint_name, column_name
    FROM dba_cons_columns
    WHERE table_name = p_tabla;
BEGIN
  dbms_output.put_line('Las restricciones de la tabla ' || p_tabla || ' son:');
  dbms_output.put_line(chr(9));
  FOR i IN c_restricciones
  LOOP
    dbms_output.put_line('Nombre: ' || i.constraint_name || ' Tipo: ' || tipo_constraint(p_tabla,i.constraint_name) || ' Columna: ' || i.column_name);
  END LOOP;
END;
/
```

```sql
EXEC mostrar_restricciones_tabla('EMP');
```

```shell
Las restricciones de la tabla EMP son:

Nombre: PK_EMP Tipo: P Columna: EMPNO
Nombre: FK_DEPTNO Tipo: R Columna: DEPTNO

PL/SQL procedure successfully completed.
```

## Ejercicio 11

Realiza al menos dos de los ejercicios anteriores en Postgres usando PL/pgSQL.

### Pasos previos

Descargo el script y lo renombro:

```shell
wget https://gist.githubusercontent.com/julianhyde/13b55f716060649da96d8dcad1316546/raw/1cca186e341ff58784ca37c95a2970f39a95622c/scott-sql-fiddle-postgresql.sql
mv scott-sql-fiddle-postgresql.sql scott-postgres.sql
```

Creo la bd:

```sql
CREATE DATABASE scott;
```

Me conecto:

```shell
\connect scott
```

Ejecuto el script para poblar la bd:

```shell
\i /home/vagrant/scott-postgres.sql
```

Creo el usuario `scott` con contraseña `tiger` en Debian y en PostgreSQL:

```shell
sudo adduser scott
create user scott with encrypted password 'tiger';
```

Le doy los privilegios necesarios:

```sql
GRANT all privileges ON DATABASE scott TO scott;
\connect scott
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO scott;
```

### Ejercicio 1 versión PostgreSQL

```sql
CREATE OR REPLACE PROCEDURE mostrar_emp_7782()
AS $$
DECLARE
  v_nombre emp.ename%TYPE;
  v_salario emp.sal%TYPE;
BEGIN
  SELECT ename, sal
  INTO v_nombre, v_salario
  FROM emp
  WHERE empno = 7782;

  RAISE NOTICE 'El nombre del empleado con codigo 7782 es % y su salario es %', v_nombre, v_salario;
END;
$$ LANGUAGE plpgsql;
```

```sql
call mostrar_emp_7782();
```

```shell
NOTICE:  El nombre del empleado con codigo 7782 es CLARK y su salario es 2450.00
CALL
```

### Ejercicio 2 versión PostgreSQL

```sql
CREATE OR REPLACE PROCEDURE nombre_empleado(p_empno emp.empno%TYPE)
AS $$
DECLARE
  v_nombre emp.ename%TYPE;
BEGIN
  SELECT ename
  INTO v_nombre
  FROM emp
  WHERE empno = p_empno;
    
  RAISE NOTICE 'El nombre del empleado con codigo % es %', p_empno, v_nombre;
END;
$$ LANGUAGE plpgsql;
```

```sql
call nombre_empleado(7876);
```

```shell
NOTICE:  El nombre del empleado con codigo 7876 es ADAMS
CALL
```
