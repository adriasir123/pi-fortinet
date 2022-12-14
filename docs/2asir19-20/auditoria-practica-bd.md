---
title: "Práctica 6: Auditoría de bases de datos"
categories: 2asir1
---

# 1. Activa desde SQL*Plus la auditoría de los intentos de acceso fallidos al sistema. Comprueba su funcionamiento. 

## Habilitamos la auditoría para los intentos de login fallidos

Para poder auditar actividad, necesitamos el privilegio de sistema `AUDIT SYSTEM`
> Con _auditar_ básicamente nos referimos a activar el registro de determinados eventos que sucedan en relación a Oracle

Como necesitamos ese permiso, activaremos la auditoría para los intentos de login fallidos como sysdba  
Este será el siguiente comando a usar para activarla
```
audit create session whenever not successful;
```

## Parámetros de inicialización

```
alter system set audit_trail=db scope=spfile sid='*';
alter system set audit_file_dest='/opt/oracle/audit/orcl/' scope=spfile sid='*';
```

Para que de verdad se almacenen los resultados de las auditorías, debemos dar un valor al parámetro de inicialización `AUDIT_TRAIL`. El valor podrá ser cualquiera, mientras que no sea _nada_, que es como se encontraba este parámetro desde la instalación de Oracle al principio. Si el parámetro no tiene un valor, no se registran los eventos de auditoría, aunque hayamos habilitado los que queramos.



## Hacemos intentos de inicio de sesión fallidos intencionados, para que se vayan auditando

Sysdba permite entrar con cualquier contraseña, así que no podemos probar éste evento usando ese rol (sys as sysdba)

En su lugar por ejemplo, voy fallar en los inicios de sesión con el usuario `HR`. Una vez fallar varias veces, utilizaremos la siguiente query para ver los resultados auditados:

```
select username, action, action_name, timestamp
    from dba_audit_trail;
```

```
USERNAME
--------------------------------------------------------------------------------
    ACTION ACTION_NAME			TIMESTAMP
---------- ---------------------------- ---------
SYSTEM
       100 LOGON			05-MAR-20

SYSTEM
       100 LOGON			05-MAR-20

SYSTEM
       100 LOGON			05-MAR-20

HR
       100 LOGON			05-MAR-20

HR
       100 LOGON			05-MAR-20

HR
       100 LOGON			05-MAR-20

HR
       100 LOGON			05-MAR-20


7 rows selected.
```
> La vista `DBA_AUDIT_TRAIL` muestra absolutamente todas las entradas de auditorías que Oracle ha registrado

Como podemos observar, los únicos eventos registrados están relacionados con hacer login (ya que es el único evento que hemos habilitado para su registro). En este caso los eventos indican fallos de login. Tienen código `100` y nombre `LOGON`



# 2. Realiza un procedimiento en PL/SQL que te muestre los accesos fallidos junto con el motivo de los mismos, transformando el código de error almacenado en un mensaje de texto comprensible.

select username, action, action_name, timestamp, returncode from dba_audit_trail;

```

CREATE OR REPLACE Function FuncionFallo(p_codigo varchar2)
return varchar2
IS
BEGIN
    CASE p_codigo
    WHEN '1017' THEN
        RETURN 'LOGIN ERROR';
    WHEN '2004' THEN
        RETURN '';
    ELSE
        RETURN 'UNKNOWN ERROR';
    END CASE;
END;

CREATE OR REPLACE Procedure failed_accesses

IS
    cursor c_audit is
    select username, action, action_name, timestamp, returncode 
    from dba_audit_trail;

    v_fallo varchar2(20);
{"usuario":"valor","usuario":"valor2"}

BEGIN

   for v_audit in c_audit loop
        v_fallo:=FuncionFallo(v_audit.returncode);
        dbms_output.put_line(v_audit.username||chr(9)||v_audit.action||chr(9)||v_audit.action_name||chr(9)||v_audit.timestamp||chr(9)||v_fallo);
    end loop;
END;
```




Return codes and messages (commonly seen)
ORA-01017: invalid username/password; logon denied
ORA-01045: user lacks CREATE SESSION privilege; logon denied
ORA-03136: inbound connection timed out
ORA-28000: the account is locked
ORA-28001: the password has expired
ORA-28003: password verification for the specified password failed
ORA-28007: the password cannot be reused
ORA-28043: invalid bind credentials for DB-OID connection



















# 3. Activa la auditoría de las operaciones DML realizadas por SCOTT. Comprueba su funcionamiento.

## Activación de la auditoría

Todas las operaciones DML existentes son:
* SELECT
* INSERT
* UPDATE
* DELETE
* MERGE
* CALL
* EXPLAIN PLAN
* LOCK TABLE

La herramienta nativa que nos ofrece Oracle para controlar la auditoría nos dejará auditar todos esos eventos, menos:
* MERGE
* EXPLAIN PLAN


Con el siguiente comando entonces, activaremos la auditoría para dichos eventos:
```
AUDIT SELECT TABLE, INSERT TABLE, UPDATE TABLE, DELETE TABLE, EXECUTE PROCEDURE, LOCK TABLE
    BY SCOTT BY ACCESS;
```
> Con audit, la opción `EXECUTE PROCEDURE` es prácticamente lo mismo que decir que estamos auditando las sentencias `CALL`.  
¿Por qué? Básicamente, porque al decir que auditamos `EXECUTE PROCEDURE`, sólo se registran eventos cuando se ejecuten órdenes `CALL` (tal y como se dice en la [documentación oficial de audit](https://docs.oracle.com/cd/B19306_01/server.102/b14200/statements_4007.htm))

Una vez que la acción se haya realizado con éxito, aparece lo siguiente:
```
Audit succeeded.
```

## Comprobación de INSERT TABLE

Añadimos el siguiente registro a una tabla `TEST` que hemos creado en SCOTT, precisamente para probar estas acciones de auditoría
```
INSERT INTO TEST
VALUES (1,'value2', 'value3'); 
```

Luego, comprobamos que se haya registrado el evento correspondiente
```
SELECT username, action, action_name, timestamp, returncode
    FROM dba_audit_trail
    WHERE action_name='INSERT' AND username='SCOTT'
    ORDER BY timestamp;
```
> Evidentemente, filtramos SÓLO los eventos de INSERT hechos por SCOTT en este caso, y ordenamos por fecha. Así nos saldrán los más nuevos al final (ordenando ascendente, como funciona por defecto)

```
USERNAME															     ACTION ACTION_NAME 		 TIMESTAMP RETURNCODE
-------------------------------------------------------------------------------------------------------------------------------- ---------- ---------------------------- --------- ----------
SCOTT																	  2 INSERT			 05-MAR-20	  984
SCOTT																	  2 INSERT			 05-MAR-20	    0
SCOTT																	  2 INSERT			 05-MAR-20	    0
SCOTT																	  2 INSERT			 10-MAR-20	    0

```
El último evento de INSERT, datado a 10 de Marzo, es el que indica la acción que hemos realizado antes, ya que es el último y único evento de INSERT en la fecha actual (realizado por SCOTT claro)

## Comprobación de UPDATE TABLE
























## Comprobación de SELECT TABLE





## Comprobación de DELETE TABLE




## Comprobación de EXECUTE PROCEDURE





## Comprobación de LOCK TABLE
































# 4. Realiza una auditoría de grano fino para almacenar información sobre la inserción de empleados del departamento 10 en la tabla emp de scott.
# 5. Explica la diferencia entre auditar una operación by access o by session.
# 6. Documenta las diferencias entre los valores db y db, extended del parámetro audit_trail de ORACLE. Demuéstralas poniendo un ejemplo de la información sobre una operación concreta recopilada con cada uno de ellos.
# 7. Localiza en Enterprise Manager las posibilidades para realizar una auditoría e intenta repetir con dicha herramienta los apartados 1, 3 y 4.
# 8. Averigua si en Postgres se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.
# 9. Averigua si en MySQL se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.
# 10.  Averigua las posibilidades que ofrece MongoDB para auditar los cambios que va sufriendo un documento.
# 11.  Averigua si en MongoDB se pueden auditar los accesos al sistema.
