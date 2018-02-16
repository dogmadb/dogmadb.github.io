# DogmaQL

*Reading time: 15min*

**Language spec: 1.0.0 (Feb 2018)**

**DogmaQL** is a database language based on the [Dogma](http://dogmalng.com) programming language and on the **SQL** database language.
Its statement set is very reduced: `delete`, `insert`, `select`, `set` and `update`, allowing to perform such data definition as data query.
It's possible that some specs allow other statements as, for example, `search` in **DogmaQL/FTS**.

Currently, the language has the following specs:

- **DogmaQL/KV** for key-value tables.
  Presented in this document.
- **DogmaQL/TS** for time-series tables.
- **DogmaQL/FTS** for full-text search tables.
- **DogmaQL/Q** for queue messaging systems.

Right now, the following is under development:

- **DogmaQL/Graph** for graph tables.

## Name spaces and tables

In **DogmaQL**, the data are stored in **tables**, containers of data objects, which we can see as rows, items, documents or objects.
The tables are known as **stores** too.

Two types of tables exist: the user tables and the system tables.
A **user table** is which the user creates.
A **system tables** represents a table which comes from factory.

The tables can be organized in **name spaces**.
These are as the **SQL** schemas.
So, for example, the `hr.employees` table can represent the `employees` table into the `hr` name space.
The following name spaces have specific aims:

- `ts`, for containing the time-series tables.
  Its access is described by the **DogmaQL/TS** spec.
- `fts` for containing the search tables.
  Its access is described by the **DogmaQL/FTS** spec.

The current system tables are the following:

- `db.tables`.
  It represents the user tables.
  We also can use the `db.stores` alias.

## User tables

Remember that a user table is which created by the user.

### Table scheme

The **table schema** defines the fields that can have the objects stored into the table.
Except the primary key fields, **DogmaQL** requires no type of scheme, that is, it is schemaless and it accepts dynamic schemas.

### Creating user tables

For creating a table, the `insert` statement must be used into the `db.tables`.
Next, an example shown to create the `users` table:

```
insert into db.tables({
  table = "users"
  key = "user"
})
```

In the `table` field, we set the table name.
And with `key`, the primary key field(s).

### Dropping user tables

To drop a table and its content, we have to use the `delete` statement from the `db.tables` table:

```
delete from db.tables
where table == "users"
```

### Altering user tables

The user table metadata can be modified with the `update` statement.
Next, we show an example for disabling the *time-to-live* feature in a user table:

```
update db.tables
where table == "users"
set ttl = false
```

## Primary key

A table requires a **primary key** (**PK**) for allowing to identify every object into a table.
Two or more objects never can have the same value for this primary key.

The key is indicated by the `key` field from the table object.
And it can be simple or composite.
A **simple primary key** is which formed by a one field.
And a **composite primary key**, by multiple fields.

When a table is defined, we have to set the fields (and their data types) that are doing its primary key.
As the textual fields are very frequent in the primary keys, if this is skipped, **DogmaQL** assumes `text`.
The primary key fields can be indicated as follows:

```
"fieldName"
{field = "name", type="name"}
```

Let's show some examples:

```
#simple
insert into db.tables({table="users", key="user"})
insert into db.tables({table="users", key={field="user", type="text"}})

#composite
insert into db.tables({table="booksandchapters", key=["book", "chapter"]})
insert into db.tables({table="booksandchapters", key=[
  {field="book", type="text"}
  {field="chapter", type="text"}
]})
```

When the key is composite, this must be indicated using a list that, remember, are specified through brackets (`[]`).

The possible types for the primary key fields are:

- `text`.
  A textual value.
- `num`.
  A numeric value.

## insert statement

The `insert` statement is used both inserting data and creating new user tables.
It a requirement that the new object to insert doesn't exist.
Its syntax is as follows:

```
insert into table(object, object, object)
```

Each object is represented by a **Dogma** map that, remember, is as follows:

```
{field=value, field=value, field=value...}
```

We are going to remember too that every field can be set by line.
In which case, the commas are optional.

Here some illustrative examples for creating two objects in the `users` table:

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

We can use `in` instead on `into` too.

**Important note**. The primary key field must be specified as first.
It's a optimization requirement for the **Redis** driver.
If we comply with this requirement, we can use the app, without modifying its queries, with other **DogmaQL** drivers.

## set statement

The `set` statement is used for setting the object associated to a key.
If the key already exists, its value object is replaced.
If not, the key-value is created.
In databases, this operation is as known as *upsert*.

Its syntax is similar to the `insert` statement:

```
set into table(object, object, object)
```

We can use `in` instead on `into` too.

Example:

```
set into users({user="me", passwd="mypass"})
```

**Important note**. Like the `insert` statement, the primary key field must be specified first.

## delete statement

To remove objects, items, documents or rows, the `delete` statement must be used:

```
delete from table
where primaryKeyValue
[if condition]
[return old]
```

The `where` clause contains the value for the primary key of the item to remove.
While the `if` clause, an additional condition that must be complied for performing the deletion indicated in the `where` clause.
It's very important that we remember that the `where` clause only can refer primary key fields.
While the `if` clause, any other.

If we need to get the removed object, the `return old` clause can be indicated.

Let's see examples:

```
delete from users
where user == "me"

delete from users
where user == "me"
if not active
return old
```

## update statement

The `update` statement is used for modifying individual fields of an existing object:

```
update table
[where primaryKeyValue]
[if condition]
set field=value, field=value...
[return old|new|{old,new}]
```

Example:

```
update users
where user == "me"
set passwd == "newpass"
```

In addtion to the `=` operator, we also can use `+=`.
Example:

```
update videos
where video == "https://www.youtube.com/watch?v=Qj6tO9qlYL0&list=RDQj6tO9qlYL0"
set views += 1
```

When needed, we can request to get:

- `old`, object value prior to the modification.
- `new`, object value once modified.

For it, we can use:

- `return old`, returning the old object.
- `return new`, returning the new object.
- `return {old, new}`, returning both.

Example:

```
update users
where user == "me"
set passwd = "newpass"
return new
#returning {user="me", passwd="mypass"}

update users
where user == "me"
set passwd = "newpass"
return {old, new}
#returning {old={user="me", passwd="mypass"}, new={user="me", passwd="mypass"}}
```

Remember the three statements to write data:

- `insert`, for inserting new objects in a table.
- `update`, for updating fields from **existing** objects.
- `set`, for replacing existing objects or to create them in otherwise.

## select statement

To get objects from a table, we use the `select` statement:

```
select from table
[where primaryKeyValue]
[if condition]
[set field += 1]
```

Like the `update` statement, we have two clauses for setting the object to get.
The primary key fields are indicated in the `where` clause.
While the remain fields, in the `if`.

On the other hand, the `set` clause allows to perform an increment of one got-object's field.
It's ideal for incrementing counter fields as, for example, the views counter, the downloads counter, etc.
Next, we show two equivalent quieries:

```
#using only one statement
select from videos
where video == "https://www.youtube.com/watch?v=ljIQo1OHkTI"
set views += 1

#using two statements
update videos
where video == "https://www.youtube.com/watch?v=ljIQo1OHkTI"
set views += 1

select from videos
where video == "https://www.youtube.com/watch?v=ljIQo1OHkTI"
```

The `set` clause is only run when the statement accesses one object, that is, this has a `where` clause.
And besides, only a counter field can be indicated.

## Important aspects of the conditions

The key-value databases are very easy to implement, compared with other types of databases.
And it so, due to their operations are simple.

It's very important to keep in mind the following:

- When a `where` clause is indicated, we are **narrowing** the search to a only one object.
  Which it is set by its primary key.
  And for this reason, their fields are always compared with the `==` operator.
  So well, the find is very very quick.

- When a `where` clause not indicated, we are performing a **table scan**, that is, the engine has to fetch **every** object stored into the table.
  For this reason, this operation shouldn't be run.
  It's more, if this type of operation is frequent, usually the key-value databases are not recommended.

To identify quick and easily the queries, that are resolved with table scans, **DogmaQL** uses the `where` clause:

- When a `where` clause indicated, no table scan used.
- When a `where` clause not indicated, table scan performed.

So well, any non-PK field must be indicated into an `if` clause.
Remember, if the queries with `where` clauses are abundant, analyze if other type of database to use.

## Built-in functions

In the `if` clauses, we can use the following built-in functions:

- `exists(fieldName) : bool`, for checking whether a field exists.
- `len(fieldName) : num`, for getting the length/size of a text or list.

## Drivers

A **driver** is a software component to interact with databases, in our case, using **DogmaQL**.
Right now, we have:

- The **Redis driver**, supporting **DogmaQL/KV**, **DogmaQL/TS**, **DogmaQL/Q** and **DogmaQL/FTS**.
- The **Amazon DynamoDB driver**, supporting **DogmaQL/KV**.
  Under development.
- The **PostgreSQL driver**, supporting **DogmaQL/KV**, **DogmaQL/TS**, **DogmaQL/Q** and **DogmaQL/FTS**.
  Under development.

## Binding parameters

In a query run by a driver, it's possible to indicate values through **parameter binds**, that is, a value specification.
These binds are indicated as follows:

```
&name
```

When a query run, in addition to its code, we also have to pass a map with the parameter names and their values.
Here's an illustrative example in **JavaScript**:

```
db.run("select from users where user == &user", {user: "me"}).then(
  (res) => { ... }
  (err) => { ... }
)

#similar to:
db.run('select from users where user == "me"').then(
  (res) => { ... }
  (err) => { ... }
)
```
