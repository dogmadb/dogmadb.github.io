# dqlsh

To facilitate the query run in interactive mode, we can use the `dqlsh` app.
This command is available through the `dqlsh` package on **NPM**:

```
npm i -g dqlsh
```

To list the options: `dqlsh help`.

To open a connection to a **Redis** instance, use the `dqlsh redis` command:

```
$ dqlsh redis
redis@localhost:6379/0>
```

With the **Redis** driver, the prompt shows:

- The driver used, in our case, `redis`.
- The database server, in our case, `localhost`.
- The server port, in our case, `6379`.
- The database number, in our case, `0`.

To get the `redis` command help, use `dqlsh redis help`.

To run the written command, end it with `;` or write an empty line:

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

To cancel the written statement, `Ctrl+C` once.

To close the shell, `Ctrl+C` twice or run the internal `.exit` command.

To list the available statements, `.help`.
While for showing a command syntax, `.help statement`.
Example:

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

Note that the shell-intern commands, and independent of **DogmaQL**, start with a dot (`.`).
