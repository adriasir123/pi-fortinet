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
