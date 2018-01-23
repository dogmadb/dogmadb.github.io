# Full-text search

*Reading time: 10min*

**DogmaQL/FTS** is the **DogmaQL** spec for search engines, whose documents are stored in a key-value store.

A **search engine** is a database system specifically developed for finding terms and phrases in big amount of data quick and easily.
**Elasticsearch**, **Solr**, **Sphinx** and **RediSearch** are some examples.
Although nowadays many databases also provide similar functionalities as, for example, **PostgreSQL**.

On the one hand, the search engines usually use the **document** term as unity that describes a data entity as, for example, a product, an organization, a persona or a web page.
In our case, a document is a data object formed by a collection of fields.

On the other hand, we also have the **indexes**, the database containers where the documents are stored.
When we insert a document into an index, what the engine does firstly is analyze it.
The **analysis** aim is extract every term that appears in the document with its locations, number of times and other info.
So, when we perform a term search, the engine can response quick and easily without querying all the documents into the table.

To be able to perform a good analysis, the document language must be set.
It's not the same to analyze a document in Spanish, Italian, English or Chinese.
Every language has particular things and therefore this must be analyzed for a specific component.

Usually, the search engines are used for:

- Searching terms or phrases in big amount of data as, for example, product catalogs.
- Auto completing.

For example, in the Panama Papers a search engine was used for performing queries in 2.6 TB.

The main features of a search engine are:

- The search queries performed by the users must be answered quickly.
- The search queries must attend situations where the exact word doesn't appear, but some synonymous or variation does.
- The search engine must be accessed by multiple drivers.
- The search engine should allow the data partition.
- The search engine should allow to analyze documents written in multiple languages.
- The indexing must be performed quickly.

Some organization that are using this type of database:
`Amazon`, `AOL`, `Apple`, `BBC`, `Bloomberg`, `Cisco`, `Dell`, `Disney`, `eBay`, `Facebook`, `Fender`, `GitHub`, `Google`, `Groupon`, `IBM`, `Instagram`, `Microsoft`, `Mozilla`, `MTV`, `NASA`, `Netflix`, `Sears`, `Telefónica`, `The Guardian`, `Ticketmaster`, `Twitter` y `Wikimedia`.

## Search tables

A **search table** stores documents for performing searches quickly.
A search table is the same as an index.

En **DogmaQL**, the tables of this type are found in the `fts` name space.
Every table in this name space is a search table.

### Types of fields

A search table can have the following types of fields:

- One **key field**, which identifies the document in the table.
- One or more **search fields**, which contain the texts to index and where to perform the searches.
- Zero, one or more additional fields with supplementary data.
  On these fields, we can't perform searches and therefore they aren't indexed.

In the search tables, the primary keys are simple and are formed by the key field.
At the moment of creating the table, we have to indicate the key field and the search field(s).

### Creating search tables

Here's an example for creating this type of table, it is the same as creating a search index:

```
insert into db.tables({
  table = "fts.courses"
  key = "course"
  search = ["title", "desc"]
})
```

The table name is set in the `table` field.
The key field in `key`.
And the search fields in `search`.

### insert statement

To insert a document in a search table, we have to use the `insert` statement.
Example:

```
insert into fts.courses({
  course = "aprende-motores-de-busqueda-con-redisearch"
  lang = "es"
  title = "Aprende motores de búsqueda con RediSearch"
  desc = "..."
})
```

**Important note**. The first field must be the key field.
Please, don't forget it.

The `lang` field is used to set the document language.
It can be:

- `ar`, Arabic.
- `da`, Danish.
- `de`, German.
- `en`, English. Default.
- `es`, Spanish.
- `fi`, Finish.
- `fr`, French.
- `hu`, Hungarian.
- `it`, Italian.
- `nl`, Dutch.
- `no`, Norwegian.
- `pt`, Portuguese.
- `ro`, Romanian.
- `ru`, Russian.
- `sv`, Swedish.
- `tr`, Turkish.
- `zh`, Chinese.

## set statement

It is similar to `insert`, but if the document is existing, it is replaced.

## update statement

It is not supported by the search engines.

## delete statement

To delete a document, we have to use the `delete` statement, indicating its key value in the `where` clause.
Only the `==` operator can be used.

The `if` clause is not supported. But the `return old` does.

Example:

```
delete from fts.courses
where course == "aprende-nosql-con-redis"
return old
```

## select statement

To get a given document, we have to use the `select` statement, indicating its key value in the `where` clause.
Example:

```
select from fts.courses
where course == "a-course"
```

The `if` and `set` clauses not supported.

## search statement

To perform searches, we have to use the `search` statement:

```
search in fts.tabla
where expresión
```

The search expression can have as many predicates as needed with some of the following syntaxes:

- `field like "text"`, the field must have the given text.
- `field not like "text"`, the field mustn't have the given text.

To join multiple predicates, use the `and` and `or` operators.

If we want to perform the search in all the search fields, without setting one behind the other, use the `any` field.

Next, some illustrative examples:

```
#search the documents that contain Redis in any search field
search in fts.courses
where any like "Redis"

#search the documents that contain Redis in the title search field
search in fts.courses
where title like "Redis"

#search the documents that contain Redis or Node.js in any search field
search in fts.courses
where any like "Redis" or any like "Node.js"
```

It's very important to understand the difference between `select` and `search`.
With `select`, we get a given document.
For this reason, we have to indicate the primary key value in its `where` clause.
While `search` is used for performing searches in the search index of the table.
