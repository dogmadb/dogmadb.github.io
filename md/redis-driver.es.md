# Driver de Redis

*Tiempo de lectura: 10min*

El *driver* de [Redis](http://redis.io) para **DogmaQL** se encuentra disponible en el paquete [`dogmaql.driver.redis`](https://www.npmjs.com/package/dogmaql.driver.redis) en **NPM**.
Se recomienda una instalación local para cada proyecto que lo use.

Se ha implementado en [Dogma](http://dogmalang.com) y se ha compilado a **JavaScript**.
A lo largo de 2018, se encontrará disponible también para **Python**.

Características:

- ***100% code coverage***.
- Soporta **DogmaQL/KV**.
- Soporta **DogmaQL/TS**.
  Aunque **Redis** no es una base de datos de series temporales, el *driver* soporta este lenguaje.
  Para ello, hace uso de conjuntos ordenados.
- Soporta **DogmaQL/TFS**.
  Requiere que la instancia tenga cargado el módulo **RediSearch**.
- Soporta **DogmaQL/Q**.
- Cuando una sentencia no tiene una equivalencia uno a uno con un comando **Redis**, se ejecuta mediante un *script* **Lua** con el comando `EVAL`.
- Utiliza promesas.
- Actualmente, sólo soporta claves principales simples, esto es, de un sólo campo.
  A excepción de las tablas de series temporales, donde, por definición, las claves principales son compuestas.

Requisitos:

- La instancia de **Redis** debe tener cargados los módulos [ReJSON](http://rejson.io) y [RediSearch](http://redisearch.io).
  Se encuentra disponible la imagen **Docker** `dogmadb/redis` con estos requisitos.

Observaciones:

- Aunque en **Redis** no existe el concepto de tabla ni nada que se le asemeje, es necesario crear las tablas.
  Y suprimirlas cuando ya no se deseen utilizar.

## Acceso al driver

Un **driver** es un componente que permite el acceso a bases de datos, en nuestro caso, mediante **DogmaQL** a **Redis**.

En primer lugar, hay que instanciar el *driver*.
Para ello, hay que importarlo e instanciarlo:

```
const driver = require("dogmaql.driver.redis");
const drv = driver();
```

## Apertura de conexión

Una vez disponemos del *driver* a usar, lo siguiente es crear una **conexión** (*connection*) a la instancia de bases de datos **Redis**.
Esto es, una comunicación entre nuestro software y el servidor.
Para ello, hay que utilizar el método `createConnection()` del *driver*:

```
createConnection() : Connection
```

Ejemplo:

```
const cx = drv.createConnection();
```

Y a continuación, no hay más que abrir la conexión mediante su método `open()`:

```
open(opts:Object) : Promise
```

Las opciones disponibles son:

- `host` (string). Servidor **Redis**.
- `port` (number). Puerto en el que escucha el servidor.
- `db` (number). Número de base de datos.
- `password` (string). Contraseña de usuario.

Ejemplo:

```
cx.open({host: "localhost", port: 6379, password: "mypass"}).then(
  () => {
    //apertura con éxito
  },

  (err) => {
    //apertura fallida
  }
);
```

Para saber si una conexión se encuentra abierta, se puede consultar su campo `opened`:

```
opened : bool
```

## Cierre de conexión

Se recomienda encarecidamente cerrar la conexión cuando ya no se vaya a usar para liberar recursos, tanto en el lado cliente como en el servidor.
Esto se hace con el método `close()` del objeto conexión:

```
close() : Promise
```

Ejemplo:

```
cx.close();
```

## Objeto db

Para acceder a la base de datos activa, contra la que hemos abierto la conexión, hay que usar el campo `db` de la conexión:

```
db : Database
```

## Ejecución de consultas

Para ejecutar consultas **DogmaQL**, se utiliza el método `run()` del objeto `db`:

```
run(cmd:string) : Promise
run(cmd:string, params:object) : Promise
```

He aquí un ejemplo de consulta no parametrizada:

```
const db = cx.db;

db.run("select from users").then(
  (res) => {
    //si ok
  },

  (err) => {
    //si error
  }
);
```

Y ahora, una consulta parametrizada:

```
db.run("select from users where username==&user", {user: "me"}).then(
  (res) => {
    //si ok
  },

  (err) => {
    //si error
  }
);
```

Cuando la consulta puede devolver varios objetos de datos, el resultado es un *array*.
En cambio, cuando sólo puede ser uno, es el propio objeto.
En caso de no obtener nada, se obtendrá `null` como resultado.
