# Redis driver

*Reading time: 10min*

The [Redis](http://redis.io) driver for **DogmaQL** is available in the **NPM** [`dogmaql.driver.redis`](https://www.npmjs.com/package/dogmaql.driver.redis) package.
Recommended a local installation for every project.

This has been implemented in [Dogma](http://dogmalang.com) and compiled to **JavaScript**.
Throughout 2018, a **Python** version is going to be available.

Features:

- **100% code coverage**.
- **DogmaQL/KV** support.
- **DogmaQL/TS** support.
  Although **Redis** is not a time-series database, the *driver* supports this spec.
  For it, it uses sorted sets.
- **DogmaQL/TFS** support.
  **RediSearch** module required.
- **DogmaQL/Q** support.
- When a **DogmaQL** statement hasn't a similar one in **Redis**, a **Lua** script is run with `EVAL`.
- Promises are used.
- Right now, it only supports simple primary keys.
  Except the time-series tables, where, by definition, the primary keys are composite.

Requirements:

- The **Redis** instance must have loaded the following modules:
  [ReJSON](http://rejson.io) and [RediSearch](http://redisearch.io).
  A **Docker** image is available with these requirements: `dogmadb/redis`.

Notes:

- Although the table concept doesn't exist in **Redis**, these must be created with the `insert` statement.
  And these must be removed when not used, with `delete`.

## Importing driver

A **driver** is a component to access databases, in our case, **Redis** via **DogmaQL**.

First, the driver must be instantiated.
For it, have to import it and instantiate it:

```
const driver = require("dogmaql.driver.redis");
const drv = driver();
```

## Opening connections

Once we have a driver instance, next we have to create a **connection** to the database instance.
That is, a communication channel from our software to the server.
For it, have to use the driver's `createConnection()` method:

```
createConnection() : Connection
```

Example:

```
const cx = drv.createConnection();
```

And next, have to open the connection using its `open()` method:

```
open(opts:Object) : Promise
```

The available options are:

- `host` (string). **Redis** server.
- `port` (number). Port where the server is listening.
- `db` (number). Database number.
- `password` (string). Access password.

Example:

```
cx.open({host: "localhost", port: 6379, password: "mypass"}).then(
  () => {
    //success
  },

  (err) => {
    //failure
  }
);
```

For knowing if a connection is opened, use the `opened` field:

```
opened : bool
```

## Closing connections

Recommended closing connections when not going to use for releasing resources in the client and the server.
This is done with the connection's `close()` method:

```
close() : promise
```

Example

```
cx.close();
```

## db object

For accessing to the active database, use the connection's `db` field:

```
db : Database
```

## Running queries

For running **DogmaQL** queries , use the `db`'s `run()` method:

```
run(cmd:string) : Promise
run(cmd:string, params:object) : Promise
```

Here's an example of non-parameterized query:

```
const db = cx.db;

db.run("select from users").then(
  (res) => {
    //ok
  },

  (err) => {
    //error
  }
);
```

And now, parameterized one:

```
db.run("select from users where username==&user", {user: "me"}).then(
  (res) => {
    //ok
  },

  (err) => {
    //error
  }
);
```

When the query can return multiple data objects, the result is an array.
Instead, when only one can be returned, the result is the own object.
In case of returning nothing, `null` is got as result.
