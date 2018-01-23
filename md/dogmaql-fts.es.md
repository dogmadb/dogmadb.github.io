# Búsqueda de texto

*Tiempo de lectura: 10min*

**DogmaQL/FTS** es la especificación de **DogmaQL** para motores de búsqueda, cuyos documentos se almacenan en un sistema clave-valor.

Un **motor de búsqueda** (*search engine*) es un sistema de gestión de bases de datos desarrollado específicamente para encontrar términos o frases en grandes cantidades de datos de manera rápida y sencilla.
Entre otros motores, encontramos **Elasticsearch**, **Solr**, **Sphinx** y **RediSearch**.
Aunque hoy en día son muchas las bases de datos que proporcionan funcionalidades similares como es el caso de **PostgreSQL**.

Los motores de búsqueda proporcionan varios conceptos.
Por un lado, suelen utilizar el término **documento** (*document*), como unidad que describe una entidad de datos como, por ejemplo, un producto, una organización, una persona o una página web.
Es importante no asociar el término al concepto de archivo como un PDF o un Word.
Podría ser así, pero no tiene por qué serlo siempre.
Podemos tener una descripción detallada de un producto, sin necesidad de estar asociada a ningún archivo, representado mediante un documento en la base de datos.
En nuestro caso, un documento es un objeto de datos formado por un secuencia de campos.

Por otra parte, tenemos los **índices** (*indexes* o *indices*), los contenedores de la base de datos donde se almacena los documentos, de manera organizada.
Cuando añadimos un documento a un índice, lo que hace el motor en primer lugar es analizarlo.
El objetivo del **análisis** (*analysis*) es extraer los términos que aparecen en el documento, sus posiciones, el número de veces y otra información.
Así cuando busquemos por un término o palabra, la búsqueda será muy rápida.
El índice tiene registrados, en una estructura interna, los términos de todos sus documentos y, como no, en qué documentos aparecen.
Cada vez que se realiza una búsqueda no se recorrerá todos los documentos, sino la estructura interna de términos mantenida por el índice.
La cual es mucho más pequeña y fácil de recorrer.

Para poder realizar un buen análisis, es necesario indicar el idioma del documento.
No es lo mismo analizar un documento en español, en italiano, en inglés o en chino.
Cada idioma tiene sus cosas y debe ser por tanto analizado por un componente específico.

Generalmente se usa motores de búsqueda para:
- Búsqueda de términos o frases en grandes cantidades de datos como, por ejemplo, en catálogos de productos.
- Autocompleción.
  Es muy habitual utilizar motores de búsqueda para encontrar términos que comienzan por un determinado texto o prefijo habitualmente usados por muchas aplicaciones webs y móviles.

Por ejemplo, en la investigación de los Papeles de Panamá se utilizó un motor de búsqueda para realizar búsquedas en los 2.6 terabytes de datos disponibles.

Las principales características de un motor de búsqueda son:

- Las consultas de búsqueda realizadas por los usuarios deben responderse rápidamente.
- Las consultas de búsqueda deben atender situaciones donde no aparezca la palabra exacta pero sí algún sinónimo o variación de la palabra.
- Se debe poder acceder al motor de búsqueda mediante distintos drivers.
- Debe de permitir el escalado de bases de datos de búsqueda grandes mediante algún tipo de partición como, por ejemplo, *sharding*.
- Para asegurar la alta disponibilidad y la recuperación ante fallos de sistema, debe permitir la replicación de datos.
- Debe ser capaz de analizar documentos escritos en varios idiomas.
- Debe realizar la indexación de manera rápida, eficiente y eficaz.

Entre las organizaciones que usan motores de búsqueda, encontramos:
`Amazon`, `AOL`, `Apple`, `BBC`, `Bloomberg`, `Cisco`, `Dell`, `Disney`, `eBay`, `Facebook`, `Fender`, `GitHub`, `Google`, `Groupon`, `IBM`, `Instagram`, `Microsoft`, `Mozilla`, `MTV`, `NASA`, `Netflix`, `Sears`, `Telefónica`, `The Guardian`, `Ticketmaster`, `Twitter` y `Wikimedia`.

## Tablas de búsqueda

Una **tabla de búsqueda** (*full-text search table* o *search table*) es aquella que almacena documentos en los que es posible realizar búsquedas de texto.
Representa un índice.

En **DogmaQL**, las tablas de este tipo se encuentran en el espacio de nombres `fts`.
Cualquier tabla almacenada en este espacio de nombres será una tabla de búsqueda de texto.

### Tipos de campos

Una tabla de búsqueda presenta los siguientes tipos de campos:

- Un **campo clave** (*key field*), aquel que identifica de manera única el documento dentro de la tabla.
- Uno o más **campos de búsqueda** (*search fields*), aquellos que contienen textos a indexar y en los que buscar.
- Cero, uno o más campos adicionales que proporcionan información suplementaria del documento.
  Sobre estos campos no se realizará ningún tipo de búsqueda y, por lo tanto, no son indexados.

En las tablas de búsqueda, las claves principales son simples y están formadas por el campo clave.
En el momento de crear la tabla, hay que indicar el campo clave y los campos de búsqueda.

### Creación de tablas de búsqueda

He aquí un ejemplo de creación de este tipo de tabla o, lo que es lo mismo, de un índice de búsqueda:

```
insert into db.tables({
  table = "fts.courses"
  key = "course"
  search = ["title", "desc"]
})
```

El nombre de la tabla se indica en el campo `table`.
La clave principal en `key`.
Y los campos de búsqueda en `search`, los cuales son siempre campos de texto.

## Sentencia insert

Para insertar un documento en una tabla de búsqueda se utiliza la sentencia `insert`.
Ejemplo:

```
insert into fts.courses({
  course = "aprende-motores-de-busqueda-con-redisearch"
  lang = "es"
  title = "Aprende motores de búsqueda con RediSearch"
  desc = "..."
})
```

**Nota importante**. El primer campo siempre debe ser el campo clave.
Los restantes son indiferentes.
Por favor, no lo olvide.

El campo `lang` se utiliza para indicar el lenguaje en el que se encuentra el texto del documento.
Puede ser cualquiera de los siguientes:

- `ar`, árabe.
- `da`, danés.
- `de`, alemán.
- `en`, inglés. El predeterminado.
- `es`, español.
- `fi`, finés.
- `fr`, francés.
- `hu`, húngaro.
- `it`, italiano.
- `nl`, holandés.
- `no`, noruego.
- `pt`, portugués.
- `ro`, rumano.
- `ru`, ruso.
- `sv`, sueco.
- `tr`, turco.
- `zh`, chino.

## Sentencia set

Es similar al `insert`, pero si ya existe reemplaza el documento.

## Sentencia update

No se permite la sentencia `update`.
Ni para modificar los metadatos de una tabla de búsqueda ni para modificar campos individuales de un documento.

## Sentencia delete

Para suprimir documentos, hay que utilizar la sentencia `delete`, indicando la clave del documento en su cláusula `where`.
Sólo puede indicar la clave principal, cuyo valor debe compararse por igualdad (`==`).

No se puede usar la cláusula `if` de **DogmaQL**. Y sí, la cláusula `return old`.

Ejemplo:

```
delete from fts.courses
where course == "aprende-nosql-con-redis"
return old
```

## Sentencia select

Para obtener un determinado documento, hay que utilizar la sentencia `select` con una cláusula `where` que contenga el valor de la clave principal.
Ejemplo:

```
select from fts.courses
where course == "a-course"
```

Las cláusulas `if` y `set` no se pueden utilizar.

## Sentencia search

Para realizar búsquedas, hay que utilizar la sentencia `search`:

```
search in fts.tabla
where expresión
```

La expresión de búsqueda puede contener tantos predicados como sea necesario de cualquiera de los siguientes formatos:

- `campo like "texto"`, el campo debe contener el texto indicado.
- `campo not like "texto"`, el campo no debe contener el texto indicado.

Para unir los predicados se puede utilizar los operadores `and` y `or`.

Si la búsqueda de un texto se quiere realizar en cualquier campo de búsqueda, sin necesidad de enumerarlos uno detrás de otro, se puede utilizar el campo `any`.

A continuación, unos ejemplos:

```
#busca los documentos que contienen el texto Redis
#en algún campo de búsqueda
search in fts.courses
where any like "Redis"

#busca los documentos que contienen el texto Redis
#en su campo de búsqueda title
search in fts.courses
where title like "Redis"

#busca los documentos que contienen Redis o Node.js
#en alguno de sus campos de búsqueda
search in fts.courses
where any like "Redis" or any like "Node.js"
```

Es importante tener clara la diferencia que hay entre `select` y `search`.
Con `select`, accedemos a un determinado documento.
Por esta razón, indicaremos su clave principal en su cláusula `where`.
En cambio, mediante `search` realizamos búsquedas de texto en todos los documentos de la tabla o índice.
