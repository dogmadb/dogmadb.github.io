# DogmaQL

*Tiempo de lectura: 15min*

**DogmaQL** es un lenguaje de bases de datos basado en el lenguaje de programación [Dogma](http://dogmalang.com) y en el lenguaje de bases de datos **SQL**.
Su conjunto de sentencias es muy reducido: `delete`, `insert`, `select`, `set` y `update`, permitiendo con ellas realizar tanto la definición como la consulta de datos.
Es posible que algunas especificaciones soporten otras sentencias como, por ejemplo, `search` en **DogmaQL/FTS**.

Actualmente, el lenguaje cuenta con las siguientes especificaciones:

- **DogmaQL/KV** para tablas clave-valor.
  Expuesto mediante el presente documento.
- **DogmaQL/TS** para bases de datos de series temporales.
- **DogmaQL/FTS** para bases de datos de búsqueda.
- **DogmaQL/Q** para sistemas de mensajería basadas en colas.

En estos momentos, se encuentra bajo desarrollo:

- **DogmaQL/Graph** para bases de datos de grafos.

## Espacios de nombres y tablas

En **DogmaQL**, los datos se almacenan en **tablas** (*tables*), contenedores de objetos de datos, los cuales pueden verse como filas, ítems, documentos u objetos.
Las tablas también se conocen formalmente como **almacenes** (*stores*).

Existe dos tipos de tablas: las de usuario y las de sistema.
Una **tabla de usuario** (*user table*) es aquella que crea el propio usuario.
Mientras que una **tabla de sistema** (*system table*) representa una del sistema.

Las tablas se pueden ubicar en **espacios de nombres** (*name spaces*).
Son como los esquemas de **SQL**.
Así, por ejemplo, la tabla `hr.employees` puede representar la tabla `employees` del espacio de nombres `hr`.
El espacio de nombres `db` contiene las tablas de sistema.
Los siguientes espacios de nombres tienen objetivos específicos:

- `ts`, contiene tablas de series temporales.
  Su acceso se describe mediante la especificación **DogmaQL/TS**.
- `fts`, contiene tablas de búsqueda.
  Su acceso se describe mediante la especificación **DogmaQL/FTS**,
- `q`, contiene colas del sistema de mensajería.
  Su acceso se describe mediante la especificación **DogmaQL/Q**.

Las tablas de sistema actuales son las siguientes:

- `db.tables`.
  Representa las tablas de usuario.
  También se puede utilizar el alias `db.stores`.

## Tablas de usuario

Recordemos que una tabla de usuario es aquella que crea el usuario.

## Esquema de la tabla

El **esquema de la tabla** (*table schema*) define los campos que puede tener un objeto almacenado en la tabla.
A excepción de la clave principal, **DogmaQL** no exige ningún tipo de esquema, es decir, acepta esquemas dinámicos.

### Creación de tablas de usuario

Para crear una tabla, se utiliza la sentencia `insert` contra la tabla `db.tables`.
A continuación, se muestra un ejemplo con el que crear la tabla `users`:

```
insert into db.tables({
  table = "users"
  key = "user"
})
```

Mediante el campo `table` se indica el nombre de la nueva tabla.
Y con `key`, el campo clave principal de la tabla.

### Supresión de tablas de usuario

Para suprimir una tabla y su contenido, no hay más que utilizar la sentencia `delete` contra la tabla `db.tables`:

```
delete from db.tables
where table == "users"
```

### Alteración de tablas de usuario

También se puede modificar los metadatos de una tabla.
Esto se consigue mediante la sentencia `update`.
A continuación, se muestra un ejemplo para desactivar la característica *time-to-live* de una tabla clave-valor:

```
update db.tables
where table == "users"
set ttl = false
```

## Clave principal

Toda tabla dispone de una **clave principal** (*primary key*), la cual representa la combinación de campos a usar para identificar de manera única un objeto, ítem, documento o fila almacenado en la tabla.
Nunca dos objetos almacenados en la misma tabla pueden tener el mismo valor de clave principal.

Esta clave se indica mediante el campo `key` del objeto tabla.
Y puede ser simple o compuesta.
Una **clave principal simple** (*simple primary key*) es aquella que está formada por un único campo.
Mientras que una **clave principal compuesta** (*composite primary key*), por varios.

Cuando se define la tabla, hay que indicar los campos y los tipos de datos que forman su clave principal.
Como son muy frecuentes los campos de texto, si se omite el tipo, se asume `text`.
Estos campos se pueden indicar de dos formas:

```
"campo"
{field = "campo", type="tipo"}
```

He aquí unos ejemplos:

```
#simple
insert into db.tables({table="users", key="user"})
insert into db.tables({table="users", key={field="user", type="text"}})

#compuesta
insert into db.tables({table="booksandchapters", key=["book", "chapter"]})
insert into db.tables({table="booksandchapters", key=[
  {field="book", type="text"}
  {field="chapter", type="text"}
]})
```

Cuando la clave es compuesta, debe indicarse siempre mediante una lista que, recordemos, se especifica mediante corchetes (`[]`).

Los posibles tipos de datos de los campos claves son:

- `text`, un valor textual.
- `num`, un valor numérico.

## Sentencia insert

La sentencia `insert` se utiliza tanto para insertar datos en una tabla como para crear nuevas tablas de usuario.
Es requisito indispensable que el nuevo objeto a insertar no exista.
Su sintaxis es como sigue:

```
insert into tabla(objeto, objeto, objeto)
```

Cada objeto se representa mediante un mapa de **Dogma**, cuya sintaxis, recordemos, es como sigue:

```
{campo=valor, campo=valor, campo=valor...}
```

Recordemos también que cuando se especifica un campo por línea, las comas son opcionales.

He aquí unos ejemplos ilustrativos con los que insertar dos objetos en la tabla `users`:

```
insert into users({user="me", passwd="mypass"}, {user="you", passwd="yourpass"})

insert into users(
  {user="me", passwd="yourpass"}
  {
    user="you"
    passwd="yourpass"
  }
)
```

También se puede usar `in`, en vez de `into`.

**Nota importante**. El campo clave principal se debe indicar **siempre** el primero.
Esto se debe a una restricción de optimización del *driver* de **Redis**.
Si la cumplimos, podremos utilizar la aplicación sin modificar sus consultas con cualquier otro *driver*.

## Sentencia set

La sentencia `set` se utiliza para establecer el objeto asociado a una clave.
Si la clave ya existe, su objeto valor se reemplaza por el nuevo.
Si no existe, lo creará.
En bases de datos, esta operación se conoce formalmente como *upsert*.

Su sintaxis es similar a la de la sentencia `insert`:

```
set into tabla(objeto, objeto, objeto...)
```

También se puede usar `in`, en vez de `into`.

Ejemplo:

```
set into users({user="me", passwd="mypass"})
```


**Nota importante**. Al igual que en `insert`, el campo clave principal se debe indicar **siempre** el primero.

## Sentencia delete

Para suprimir objetos, se utiliza la sentencia `delete`, cuya sintaxis es la siguiente:

```
delete from tabla
where valorClave
[if condición]
[return old]
```

La cláusula `where` contiene el valor clave del objeto a suprimir.
Mientras que la cláusula `if`, una condición adicional que debe cumplirse para llevar a cabo la supresión del objeto indicado en el `where`.
Es muy importante que recuerde que el `where` sólo puede referenciar los campos de la clave principal.
Mientras que el `if`, cualquier otro.

Si deseamos recuperar el objeto suprimido, se puede indicar la cláusula `return old`.

He aquí unos ejemplos ilustrativos:

```
delete from users
where user == "me"

delete from users
where user == "me"
if not active
return old
```

## Sentencia update

La sentencia `update` se utiliza para actualizar campos de un objeto ya existente:

```
update tabla
where valorClave
[if condición]
set campo=valor, campo=valor...
[return old|new|{old,new}]
```

Ejemplo:

```
update users
where user == "me"
set passwd == "newpass"
```

Además del operador `=` con el que indicar el nuevo valor de un campo, tambien se puede utilizar `+=`.
Ejemplo:

```
update videos
where video == "https://www.youtube.com/watch?v=Qj6tO9qlYL0&list=RDQj6tO9qlYL0"
set views += 1
```

Si se desea, se puede solicitar que se devuelva:

- `old`, valor del objeto previo a la actualización.
- `new`, valor del objeto tras la actualización.

En caso de hacerlo, he aquí las posibilidades:

- `return old`, devuelve el objeto previo a la actualización.
- `return new`, devuelve el objeto tras la actualización.
- `return {old, new}`, devuelve un objeto con dos campos: `old` y `new`.

He aquí unos ejemplos ilustrativos:

```
update users
where user == "me"
set passwd = "newpass"
return new
#devuelve: {user="me", passwd="mypass"}

update users
where user == "me"
set passwd = "newpass"
return {old, new}
#devuelve: {old={user="me", passwd="mypass"}, new={user="me", passwd="mypass"}}
```

Recordemos las tres sentencias de escritura:

- `insert` inserta un **nuevo** objeto.
  No puede existir ninguno con la clave especificada.
- `update` actualiza campos individuales de un objeto **ya** existente.
  Si no existe, no hará nada.
- `set` cambia el objeto almacenado si ya existe o lo crea si no existe.

## Sentencia select

Para obtener objetos de una tabla, se utiliza la sentencia `select`, cuya sintaxis es como se muestra a continuación:

```
select from tabla
[where valorClave]
[if condición]
[set campo += 1]
```

Al igual que la sentencia `update`, existe dos cláusulas que restringen lo que obtener.
Las referencias a los campos de la clave principal se indican en la cláusula `where`.
Los demás, en el `if`.

Por su parte, la cláusula `set` permite realizar un incremento de uno de los campos del objeto recuperado.
Es ideal para incrementar campos de tipo contador como, por ejemplo, el contador de visitas, de visualizaciones, de descargas, etc.
He aquí dos consultas equivalentes:

```
#mediante una única sentencia
select from videos
where video == "https://www.youtube.com/watch?v=ljIQo1OHkTI"
set views += 1

#mediante dos sentencias
update videos
where video == "https://www.youtube.com/watch?v=ljIQo1OHkTI"
set views += 1

select from videos
where video == "https://www.youtube.com/watch?v=ljIQo1OHkTI"
```

La cláusula `set` sólo se ejecuta cuando el acceso se realiza contra un único objeto, el indicado por el valor clave del `where`.
Y sólo puede hacer referencia a un único campo, el cual debe incrementarse en una única unidad.

### Aspectos importantes de la sentencia select

Las bases de datos clave-valor son muy sencillas de implementar, comparado con otros tipos de bases de datos.
Y son así de simples, por su sencillo funcionamiento.

Es muy importante tener en cuenta los siguientes aspectos:

- Cuando se indica una cláusula `where`, lo que se está haciendo es acotar la búsqueda a un único objeto.
  Siempre debe indicar el valor de una clave principal.
  Y por esta razón, sus campos **siempre** se comparan por igualdad (`==`).
  Lo que conlleva que su acceso sea muy rápido y consuma pocos recursos.

- Cuando no se indica una cláusula `where`, se está haciendo un **escaneo de tabla** (*table scan*).
  Esto quiere decir que se recorre todos los objetos almacenados en la tabla.
  Por esta razón, no se recomienda este tipo de consultas.
  Pues podrían consumir muchos recursos.
  Es más, si este tipo de consultas son frecuentes, generalmente se desaconseja la utilización de bases de datos clave-valor.

Para identificar rápida y fácilmente las consulas que se resuelven con escaneos de tabla, **DogmaQL** usa la cláusula `where`:

- Si se indica una cláusula `where`, no se realiza escaneo de tabla.
- Si no se indica cláusula `where`, la consuklta se resolverá con un escaneo de tabla.

Así pues, cualquier campo que no forme parte de la clave principal tiene que ser indicado en la cláusula `if`.

Recuerde, si abundan las consultas sin `where`, analice concienzudamente si usar una base de datos clave-valor o mejor otro tipo.

## Funciones predefinidas (built-in functions)

En las cláusulas `if`, se puede usar las siguientes funciones predefinidas:

- `exists(fieldName) : bool`, para comprobar si un campo existe.
- `len(fieldName) : num`, para obtener el tamaño de un texto o lista.

## Drivers

Un **driver** es un componente de software que permite interactuar con bases de datos, en nuestro caso, mediante **DogmaQL**.
En estos momentos, disponemos de:

- El **driver de Redis** (*Redis driver*), el cual soporta **DogmaQL/KV**, **DogmaQL/TS**, **DogmaQL/Q** y **DogmaQL/FTS**.
- El **driver de Amazon DynamoDB** (*Amazon DynamoDB driver*), que implementa el lenguaje de consulta **DogmaQL/KV**.
  Bajo desarrollo.
- El **driver de PostgreSQL** (*PostgreSQL driver*), el cual soporta **DogmaQL/KV**, **DogmaQL/TS**, **DogmaQL/Q** y **DogmaQL/FTS**.
  Bajo desarrollo.

### Enlaces de parámetros

En una consulta ejecutada por un *driver*, es posible indicar valores mediante **enlaces de parámetros** (*parameter binds*), esto es, una especificación de valor.
Estos enlaces se indican mediante la siguiente sintaxis:

```
&nombre
```

Cuando se ejecute la consulta, además de pasar su código, también hay que pasar un objeto formado por los nombres de los enlaces y sus valores.
He aquí un ejemplo ilustrativo en **JavaScript**:

```
db.run("select from users where user == &user", {user: "me"}).then(
  (res) => { ... }
  (err) => { ... }
)

#similar a:
db.run('select from users where user == "me"').then(
  (res) => { ... }
  (err) => { ... }
)
```
