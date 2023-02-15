# Examen prácticas 5 y 6

## Ejercicio 1

### 1.1 Enunciado

Realiza un procedimiento para ORACLE que reciba el nombre de un rol y devuelva exclusivamente los privilegios provenientes de los otros roles que contenga, teniendo en cuenta todos los posibles niveles de anidamiento. El formato será el indicado a continuación:

Rol analizado: xxxxxxxxxxxxx

Privilegios sobre objetos:

	Rol 1: xxxxxxxxxx
		Priv1	Objeto1
		…
		PrivN	ObjetoN
		Rol 1.1: xxxxxxxxxxxxx
			Priv1	Objeto1
			…
			PrivN	ObjetoN
		Rol 1.N: xxxxxxxxxxxxx
			Priv1	Objeto1
			…
			PrivN	ObjetoN
			Rol 1.N.1: xxxxxxxxxxxx
				….
	…
	Rol N: xxxxxxxxxx
		Priv1	Objeto1
		…
		PrivN	ObjetoN

Privilegios de Sistema: 

	Rol 1: xxxxxxxxxx
		Priv1
		…
		PrivN
	Rol 1.1: xxxxxxxxxxxxx
			Priv1	Objeto1
			…
			PrivN	ObjetoN
		Rol 1.N: xxxxxxxxxxxxx
			Priv1	Objeto1
			…
			PrivN	ObjetoN
			Rol 1.N.1: xxxxxxxxxxxx
				….
	…
	Rol N: xxxxxxxxxx
		Priv1
		…
		PrivN
		







## Ejercicio 2

### 2.1 Enunciado

Realiza un procedimiento para ORACLE que reciba el nombre de un usuario y muestre la cantidad de bloques de datos de la que dispone en cada uno de los tablespaces teniendo en cuenta la cuota asignada en cada uno de los mismos. En caso de tener cuota ilimitada en alguno de ellos se mostrará el número de bloques de datos en el espacio disponible en el tablespace



















## Ejercicio 3

### 3.1 Enunciado

(corrección in situ) Crea tres usuarios en MongoDB. Crea dos colecciones Examen1 y Examen2 con dos documentos cada una con los atributos que desees. El primer usuario no podrá acceder a ninguna de las dos colecciones. El segundo podrá acceder solo a una de ellas y solo para leer los documentos. El tercer usuario tendrá acceso total a las dos colecciones. Demuestra al profesor in situ el correcto funcionamiento de los permisos

### 3.2 Realización

Creo la base de datos:

```sql
test> use examen56
switched to db examen56
examen56> 
```

Creo las colecciones:

```sql
examen56> db.createCollection("examen1")
{ ok: 1 }
examen56> db.createCollection("examen2")
{ ok: 1 }
```

Inserto los documentos:

```sql
db.examen1.insertMany( [
  { gato: "Garfield", raza: "Abinisio" },
  { gato: "Kitty", raza: "Bengala" }
] )

db.examen2.insertMany( [
  { perro: "Toby", raza: "Leonberger" },
  { perro: "Rocky", raza: "Chihuahua" }
] )
```

Creo el primer usuario, que no podrá acceder a ninguna de las dos colecciones.

Para ello primero creo `role1`:

```sql
db.createRole({
   role: "role1",
   privileges: [
      { resource: { db: "examen56", collection: "" }, actions: ["remove"] }
   ],
   roles: []
})
```

Y creo `usuario1` asignándoselo:

```sql
db.createUser(
  {
    user: "usuario1",
    pwd: "1234",
    roles: ["role1"]
  }
)
```

Me conecto como `usuario1` y hago las comprobaciones:

```sql
vagrant@servidormongodb:~$ mongosh -u usuario1 -p 1234 10.0.3.2/examen56
Current Mongosh Log ID: 63ec9f509bba6d0cab76fd60
Connecting to:          mongodb://<credentials>@10.0.3.2:27017/examen56?directConnection=true&appName=mongosh+1.6.0
Using MongoDB:          5.0.13
Using Mongosh:          1.6.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

examen56> db.examen1.find()
MongoServerError: not authorized on examen56 to execute command { find: "examen1", filter: {}, lsid: { id: UUID("be22e913-4435-4b5d-9a23-0593bcc659e0") }, $db: "examen56" }
examen56> db.examen2.find()
MongoServerError: not authorized on examen56 to execute command { find: "examen2", filter: {}, lsid: { id: UUID("be22e913-4435-4b5d-9a23-0593bcc659e0") }, $db: "examen56" }
```

Creo el segundo usuario, que podrá acceder solo a una de las colecciones y solo para leer los documentos.

Para ello primero creo `role2`:

```sql
db.createRole({
   role: "role2",
   privileges: [
     {
       resource: { db: "examen56", collection: "examen1" },
       actions: [ "find" ]
     }
   ],
   roles: []
})
```

Y creo `usuario2` asignándoselo:

```sql
db.createUser(
  {
    user: "usuario2",
    pwd: "1234",
    roles: ["role2"]
  }
)
```

Me conecto como `usuario2` y hago las comprobaciones:

```sql
vagrant@servidormongodb:~$ mongosh -u usuario2 -p 1234 10.0.3.2/examen56
Current Mongosh Log ID: 63eca1d7ecb63d1addbe39ac
Connecting to:          mongodb://<credentials>@10.0.3.2:27017/examen56?directConnection=true&appName=mongosh+1.6.0
Using MongoDB:          5.0.13
Using Mongosh:          1.6.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

examen56> db.examen1.find()
[
  {
    _id: ObjectId("63ec922d8d5f87224376c6e7"),
    gato: 'Garfield',
    raza: 'Abinisio'
  },
  {
    _id: ObjectId("63ec922d8d5f87224376c6e8"),
    gato: 'Kitty',
    raza: 'Bengala'
  }
]
examen56> db.examen2.find()
MongoServerError: not authorized on examen56 to execute command { find: "examen2", filter: {}, lsid: { id: UUID("e5171e94-e8b0-4904-9c19-966ff8aa54ed") }, $db: "examen56" }
```

Creo `usuario3`, que tendrá acceso total a las dos colecciones:

```sql
db.createUser(
  {
    user: "usuario3",
    pwd: "1234",
    roles: [ "dbOwner" ]
  }
)
```

Me conecto como `usuario3` y hago las comprobaciones:

```sql
vagrant@servidormongodb:~$ mongosh -u usuario3 -p 1234 10.0.3.2/examen56
Current Mongosh Log ID: 63eca238c508efcfa3ff9652
Connecting to:          mongodb://<credentials>@10.0.3.2:27017/examen56?directConnection=true&appName=mongosh+1.6.0
Using MongoDB:          5.0.13
Using Mongosh:          1.6.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

examen56> db.examen1.find()
[
  {
    _id: ObjectId("63ec922d8d5f87224376c6e7"),
    gato: 'Garfield',
    raza: 'Abinisio'
  },
  {
    _id: ObjectId("63ec922d8d5f87224376c6e8"),
    gato: 'Kitty',
    raza: 'Bengala'
  }
]
examen56> db.examen2.find()
[
  {
    _id: ObjectId("63ec923e8d5f87224376c6e9"),
    perro: 'Toby',
    raza: 'Leonberger'
  },
  {
    _id: ObjectId("63ec923e8d5f87224376c6ea"),
    perro: 'Rocky',
    raza: 'Chihuahua'
  }
]
```

























## Ejercicio 4

### 4.1 Enunciado

Queremos cambiar de ubicación un tablespace de Postgres, pero antes debemos avisar a los usuarios que tienen acceso de lectura o escritura a cualquiera de los objetos almacenados en el mismo. Escribe una consulta que obtenga un listado con los nombres de dichos usuarios






