# PLSQL Apuntes

## Ejercicio 5 

Recibe un nombre de usuario y devuelve su tablespace por defecto (DBA_USERS):

```sql
CREATE OR REPLACE PROCEDURE mostrar_defts(p_usuario DBA_USERS.USERNAME%TYPE)
IS
  v_nombrets  DBA_USERS.DEFAULT_TABLESPACE%TYPE;
BEGIN
    SELECT default_tablespace into v_nombrets
    FROM dba_users
    WHERE username = p_usuario;
    dbms_output.put_line('El usuario '||p_usuario||' usa por defecto '||vnombrets);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    dbms_output.put_line('El usuario '||p_usuario||' no existe');
END mostrar_defts;

CREATE OR REPLACE PROCEDURE principal
IS
BEGIN
  mostrar_defts('RAUL');
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    dbms_output.put_line('El usuario RAUL no existe');
END;



Lo errores se propagan para arriba


CREATE OR REPLACE PROCEDURE mostrar_defts(p_usuario DBA_USERS.USERNAME%TYPE)
IS
  v_nombrets  DBA_USERS.DEFAULT_TABLESPACE%TYPE;
BEGIN
    SELECT dname, loc into v_nombrets
    FROM dba_users
    WHERE username = p_usuario;
    dbms_output.put_line('El usuario '||p_usuario||' usa por defecto '||vnombrets);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    dbms_output.put_line('El usuario '||p_usuario||' no existe');
END mostrar_defts;




CREATE OR REPLACE FUNCTION devolver_sal (p_empno emp.empno%TYPE)
RETURN NUMBER
IS
  v_salario emp.sal%type;
BEGIN
  select sal into v_salario
  FROM emp
  WHERE empno=p_empno;
  return v_salario;
  -- AQUI YA NO SE EJECUTA NADA
EXCEPTION
  when no_data_found then
    dbms_output...
    return -1;
END devolver_sal;


El return es la última sentencia que se debe ejecutar, y debe tener 1 solo return


-- Sin FOR de cursor

CREATE OR REPLACE PROCEDURE mostrar_empleados (p_deptno emp.empno%TYPE)
IS
  cursor c_empleados is
  select ename
  from emp
  where deptno=p_deptno;

  v_nomemp emp.ename%type;
  e_deptsinempleados exception;

BEGIN
  comprobar_excepciones(p_deptno);
  open c_empleados;
  fetch c_empleados into v_nomemp;
  while c_empleados%FOUND AND c_empleados%ROWCOUNT>=3 loop
    procesar_empleado(v_nomemp);
    fetch c_empleados into v_nomemp;
  end loop;
  if c_empleados%ROWCOUNT=0 then
    raise e_deptsinempleados;
  end if;
  close c_empleados;
EXCEPTION
  when e_deptsinempleados then
    dbms_output.put_line('El departamento '||p_deptno||' no tiene empleados');
END;

Estructura para leer cualquier conjunto de información:

Abrir la fuente de información
Leer primer dato
Mientras la lectura sea correcta
  Procesar dato
  Leer siguiente dato
Fin Mientras
Mostrar resultados o seguir con otra cosa



for y while para recorrer un cursor




CREATE OR REPLACE PROCEDURE comprobar_excepciones (p_deptno dept.deptno%TYPE)
IS
  v_hayempleados NUMBER;
BEGIN
  select count(*) into v_hayempleados
  from emp
  where deptno=p_deptno;
  if v_hayempleados=0 then
    raise_application_error(-20001, 'El departamento '||p_deptno||' no tiene empleados');
  end if;
end;

raise_application_error se considera que el programa falla, y la ejecución se parará

raise en el bloque de excepciones se trata como un error pero la ejecución sigue

debajo del close del cursor no se puede escribir


for c_empleados into v_nomemp loop
  procesar_empleado(v_nomemp);
end loop;


%ROWCOUNT NO dice las líneas en total, es el número de filas leídas hasta ese momento

(si quieres mostrar los 3 primeros o algo así)

CREATE OR REPLACE PROCEDURE recorrer_empleados (p_deptno emp.empno%TYPE)
IS
  cursor c_empleados is
  select ename, sal, comm, deptno
  from emp
  where deptno=p_deptno;

  v_empleado c_empleados%ROWTYPE;
  e_deptsinempleados exception;

BEGIN
  comprobar_excepciones(p_deptno);
  for v_empleados in c_empleados loop
    procesar_empleado(v_empleado.ename, v_empleado.comm, v_empleado.deptno);
  end loop;
END;

CREATE OR REPLACE PROCEDURE procesar_empleados (p_emp varchar2)

con el for no hay fetch


capturar la excepción es ponerle un when

Tipos de excepciones, con nombre o sin nombre ora-12560



exception
  when others then
    case sqlcode
      when -2292 then
        ...
      when -12768 then
      asds
lo último que va a haber en el bloque de exception

update y tal no levanta excepciones tengo que hacer sql not found
```

## Triggers

Before: algún tipo de comprobación sobre inserción de datos...etc

Vista actualizable: si contiene todos los campos obligatorios de las tablas con las que se creó la vista

Eventos que hacen ejecutar un trigger: Insert, update, delete, combinación

Tipos de dispario:

- Por sentencia: impedir que alguien meta cosa a partir de las 15:00 (los datos a insertar no son importantes)
- Por fila: 

No pueden haber commits y rollback en los triggers



```sql
select 'HOLA '||ename
from emp;



select 'CREATE USER '||ename||'IDENTIFIED BY '||ename||';'
from emp




```

Triggers de sistema: after logon, before logoff, insert...


## 21/11/2022

Si quiero que un usuario sólo pueda ver determinadas filas de una tabla, tengo que crear una vista con las filas que quiero que vea.
Sobre esa tabla le doy permisos de select a ese usuario.




