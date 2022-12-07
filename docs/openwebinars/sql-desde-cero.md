# SQL desde Cero

## Pasos previos

En `servidormariadb` descargo el comprimido de `sakila`:

```shell
wget https://downloads.mysql.com/docs/sakila-db.tar.gz
```

Extraigo y entro:

```shell
tar -xvf sakila-db.tar.gz
cd sakila-db
```

Entro como root a mariadb:

```shell
sudo mariadb
```

Ejecuto los 2 scripts necesarios:

```sql
source sakila-schema.sql;
source sakila-data.sql;
```

Compruebo que tengo las tablas:

```sql
SHOW FULL TABLES;
```

Compruebo que tengo registros en `film` y en `film_text`:

```sql
SELECT COUNT(*) FROM film;
SELECT COUNT(*) FROM film_text;
```

## DML

### Modificadores

DISTINCT elimina repetidos:

```sql
select distinct last_name from actor;
```

Si empezamos a pedir datos de más columnas, DISTINCT tiene en cuenta TODAS las columnas así que habrán menos repetidos:

```sql
select distinct last_name, first_name from actor;
```

Toma en cuenta que no se repitan registros iguales de 2 columnas, lo cual será un poco más difícil.

Si hacemos lo siguiente nunca afectará DISTINCT, porque no deberían de haber registros completos repetidos nunca:

```sql
select distinct * from actor;
```

Alias de columnas, para cambiar los nombres reales de las columnas a los que queramos:

```sql
select title AS 'FILM NAME', description AS 'SINOPSIS' from film LIMIT 10;
```

Si el alias es de 1 palabra, no necesita comillas, pero si es de más de 1 palabra, sí.

### WHERE y ORDER BY

Mayor o igual:

```sql
select * from film where length >= 120;
```

Con un comentario:

```sql
select * from film /*where length >= 120*/;
```

Orden descendente (el ascendente es el por defecto):

```sql
select title from film ORDER BY title DESC;
```

Mezcla de ambos:

```sql
SELECT last_name, first_name
FROM actor
WHERE last_name = 'ALLEN'
ORDER BY first_name DESC;
```

Se puede ordenar por un campo que no se muestre:

```sql
SELECT actor_id, first_name
FROM actor
ORDER BY last_name DESC;
```

### Operadores lógicos: AND, OR, NOT, LIKE, IN, BETWEEN

Ejemplo de AND:

```sql
SELECT title, length, rental_rate
FROM film
WHERE length > 90 AND rental_rate <= 3;
```

Todas las condiciones se deben cumplir

Ejemplo de OR:

```sql
SELECT title, length, rental_rate
FROM film
WHERE length > 90 OR rental_rate <= 3;
```

El resultado suele ser mayor porque es menos estricto, o una cosa o la otra

Ejemplo de NOT:

```sql
SELECT *
FROM actor
WHERE NOT last_name = 'ALLEN';
```

La ventaja del NOT es que se puede negar un bloque entero de WHERE

Equivalente a NOT, el <> que significa diferente a:
  
```sql
SELECT *
FROM actor
WHERE last_name <> 'ALLEN';
```

Ejemplo de LIKE:

```sql
SELECT title
FROM film
WHERE title LIKE '%LOVE%';
```

Cualquier cosa que contenga LOVE

Que empiece por LOVE:

```sql
SELECT title
FROM film
WHERE title LIKE 'LOVE%';
```

Que termine por LOVE:

```sql
SELECT title
FROM film
WHERE title LIKE '%LOVE';
```

Ejemplo de BETWEEN:

```sql
SELECT rental_rate
FROM film
WHERE rental_rate BETWEEN 0.5 AND 2.5;
```

### Consultas de unión (no mismo que JOIN)

```sql
SELECT last_name, first_name
FROM customer
union
SELECT last_name, first_name
FROM actor;
```

El número de columnas tiene que ser igual, pero el nombre de ellas no influye.  
Genera una sola tabla sumando resultados  
Se tienen que mezclar campos compatibles, no se puede mezclar campos de texto con número  
Los Alias de columna solo deben estar en la primera select

```sql
SELECT last_name AS apellido, first_name AS Nombre
FROM customer
union
SELECT last_name, first_name
FROM actor;
```

Por defecto UNION no muestra repetidos

Para que muestre también repetidos hacemos:

```sql
SELECT last_name AS apellido, first_name AS Nombre
FROM customer
UNION ALL
SELECT last_name, first_name
FROM actor;
```

Para ordenar con UNION va al final:

```sql
SELECT last_name AS apellido, first_name AS Nombre
FROM customer
UNION ALL
SELECT last_name, first_name
FROM actor
ORDER BY apellido;
```

### Subconsultas

Máxima de subconsultas: siempre tienen que trabajar sobre 1 solo campo, porque luego ese campo es el que se utilizará en un filtro WHERE

Las subconsultas pueden tener más subconsultas anidadas

### Consultas con funciones escalares

Ejemplo LCASE (convierte a minúsculas):

```sql
SELECT LCASE(title)
FROM film;
```

Ejemplo CONCAT (sólo acepta texto):

```sql
SELECT CONCAT(last_name, ', ',first_name)
FROM actor;
```

Anido:

```sql
SELECT CONCAT(last_name, ', ',LCASE(first_name))
FROM actor;
```

Puedo cambiar valores usando operadores matemáticos:

```sql
SELECT title AS 'Titulo', rental_rate * 0.75 AS 'Precio euros'
FROM film;
```

### Funciones de agregado

Ejemplo count:

```sql
SELECT count(*) AS 'Num Clientes'
FROM customer
WHERE active = 1;
```

Ejemplo max:

```sql
SELECT MAX(rental_rate) AS 'Precio maximo'
FROM film;
```

Ejemplo min:

```sql
SELECT min(rental_rate) AS 'Precio minimo'
FROM film;
```

Mezcla de MAX y MIN:

```sql
SELECT MAX(rental_rate) AS 'Precio maximo', MIN(rental_rate) AS 'Precio minimo'
FROM film;
```

### Consultas de agrupación (GROUP BY / HAVING)

```sql
SELECT COUNT(film_id), rental_rate
FROM film
GROUP BY rental_rate
;
```

Normalmente GROUP BY se usa junto con funciones de agregado como COUNT

No tiene mucho sentido pedir detalle (más columnas) y a la vez pedir agrupación

Se le puede añadir WHERE y ORDER BY:

```sql
SELECT COUNT(film_id), rental_rate
FROM film
WHERE replacement_cost <= 20
GROUP BY rental_rate
ORDER BY rental_rate DESC
;
```

Ejemplo de HAVING:

```sql
SELECT COUNT(film_id), rental_rate
FROM film
WHERE replacement_cost <= 20
GROUP BY rental_rate
HAVING COUNT(film_id) > 165
ORDER BY rental_rate DESC
;
```

Hay quien llama a HAVING 'El WHERE dentro del WHERE'

Una vez que se ha agrupado, HAVING filtra los grupos

### Consultas sobre más de una tabla: JOIN (parte 1)

```sql
SELECT first_name, last_name, a.address
FROM customer c, address a
WHERE c.address_id = a.address_id
AND active = 1
;
```

### Consultas sobre más de una tabla: JOIN (parte 2)

Ejemplo INNER JOIN:

```sql
SELECT first_name, last_name, a.address
FROM customer c
INNER JOIN address a
ON c.address_id = a.address_id
;
```

Ejemplo LEFT JOIN:

```sql
SELECT first_name, last_name, a.address
FROM customer c
LEFT JOIN address a
ON c.address_id = a.address_id
;
```

Ejemplo RIGHT JOIN:

```sql
SELECT first_name, last_name, a.address
FROM customer c
RIGHT JOIN address a
ON c.address_id = a.address_id
;
```
