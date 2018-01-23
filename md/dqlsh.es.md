# dqlsh

Para facilitar la ejecución de consultas **DogmaQL** en modo interactivo, se puede utilizar el comando `dqlsh`.
El cual se encuentra diponible a través paquete **NPM** `dqlsh`.
Para una instalación global:

```
npm i -g dqlsh
```

Para listar las opciones: `dqlsh help`.

Para abrir una conexión a una instancia **Redis**, usar el comando `dqlsh redis`:

```
$ dqlsh redis
redis@localhost:6379/0>
```

Con el *driver* de **Redis**, el *prompt* indica:

- El *driver*, en este caso, `redis`.
- El servidor de base de datos, en este caso, `localhost`.
- El puerto del servidor de bases de datos, en este caso, `6379`.
- El número de base de datos, en este caso, `0`.

Para obtener la ayuda del comando `redis`, usar `dqlsh redis help`.

Para solicitar la ejecución del comando introducido, dejar una línea en blanco o finalizar el comando en `;`:

```
redis@localhost:6379/0> select from users
...
ResultSet [
  { user: 'me', passwd: 'mypass' },
  { user: 'you', passwd: 'yourpass' } ]
redis@localhost:6379/0> delete from users
... where user == "me"
... return old
...
{ user: 'me', passwd: 'mypass' }
redis@localhost:6379/0> select from users;
ResultSet [ { user: 'you', passwd: 'yourpass' } ]
redis@localhost:6379/0>
```

Para descartar el comando introducido, pulsar una vez `Ctrl+C`.

Para cerrar el *shell*, pulse dos veces `Ctrl+C` o ejecute el comando interno `.exit`.

Para listar los comandos disponibles, `.help`.
Mientras que para mostrar la sintaxis de un comando **DogmaQL**, `.help comando`.
Ejemplo:

```
redis@localhost:6379/0> .help
insert  Insert a new object.
set     Set an object.
update  Update object fields.
delete  Delete an object.
select  Select objects.
search  Search in a table.
redis@localhost:6379/0> .help delete
delete from [namespace.]table
where key
[if condition]
[return old]
redis@localhost:6379/0>
```

Observe que los comandos internos del *shell*, independientes de **DogmaQL**, comienzan por punto (`.`).
