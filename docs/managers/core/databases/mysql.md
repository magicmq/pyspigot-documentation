# MySQL

PySpigot's database manager supports connecting to MySQL and other SQL-compatible databases (MariaDB, PostgreSQL, etc.) via [HikariCP](https://github.com/brettwooldridge/HikariCP), a high-performance JDBC connection pool.

For instructions on importing the database manager into your script, visit the [General Information](../../usage.md) page.

???+ info

    This is not a comprehensive guide to MySQL or SQL. Please seek out appropriate tutorials and documentation if you are unfamiliar with SQL.

## Database Manager Usage

The following functions are available on the database manager for connecting to an SQL database:

- `newHikariConfig()`: Returns a new `HikariConfig` object with default settings. Useful if you want to customize connection pool options before connecting.
- `connectSql(host, port, database, username, password)`: Connects to a remote SQL database using the provided credentials. This is the simplest way to connect.
- `connectSql(host, port, database, username, password, hikariConfig)`: Connects to a remote SQL database using the provided credentials and a custom `HikariConfig`.
- `connectSql(uri)`: Connects using a [JDBC connection string](https://learn.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url), which allows fine-grained control over connection parameters and options.
- `connectSql(uri, hikariConfig)`: Connects using a JDBC connection string and a custom `HikariConfig`.
- `connectSql(hikariConfig)`: Connects using only a `HikariConfig` object. The HikariConfig must have its JDBC URL, username, and password set directly.
- `disconnect(database)`: Disconnects and closes an open SQL database connection. Accepts the `SqlDatabase` object returned by any `connectSql` function.

All `connectSql` functions return an `SqlDatabase` object, which is the interface used to run queries and updates against the database. Which function you use depends on your situation — `connectSql(host, port, database, username, password)` is the most straightforward for general use.

???+ tip

    You do not need to call `disconnect` when your script is stopped or unloaded — PySpigot will automatically close all open connections. However, if you are done with a connection before the script stops, it is good practice to close it manually.

## The HikariConfig

The `HikariConfig` is a configuration object that gives you finer control over the underlying connection pool. It allows you to set pool size, idle timeout, connection timeout, and many other options. For a full list, see the [HikariCP JavaDocs](https://www.javadoc.io/doc/com.zaxxer/HikariCP/latest/com.zaxxer.hikari/com/zaxxer/hikari/HikariConfig.html).

Use `newHikariConfig()` to get a new `HikariConfig` with default settings:

``` py linenums="1"
import pyspigot as ps

db_manager = ps.database_manager()

config = db_manager.newHikariConfig()
config.setMaximumPoolSize(10)
config.setConnectionTimeout(30000)

sql = db_manager.connectSql('localhost', '3306', 'my_database', 'root', 'password', config)
```

You may also use the `HikariConfig` alone to establish a connection by setting the JDBC URL, username, and password directly on the config object, and then calling `connectSql(hikariConfig)`:

``` py linenums="1"
config = db_manager.newHikariConfig()
config.setJdbcUrl('jdbc:mysql://localhost:3306/my_database')
config.setUsername('root')
config.setPassword('password')

sql = db_manager.connectSql(config)
```

## The SqlDatabase Object

When any `connectSql` function succeeds, it returns an `SqlDatabase` object. This is what you use to interact with the database:

- `select(sql)`: Executes a SELECT statement and returns the results as a list of dicts. Each element in the list is a dict representing one row, where the keys are column names and the values are the corresponding column data.
- `select(sql, values...)`: Same as above, but with parameterized values that replace `?` placeholders in the SQL statement. Pass values as a list, e.g. `[10, 'hello']`.
- `update(sql)`: Executes an INSERT, UPDATE, DELETE, or DDL statement. Returns the number of rows affected.
- `update(sql, values...)`: Same as above, with parameterized values.
- `getHikariDataSource()`: Returns the underlying `HikariDataSource` object for advanced use.

### Select Results

The `select` function returns a **list of rows**. Each row is a dict-like object where keys are column names and values are the data in that column for that row:

``` py linenums="1"
data = sql.select('SELECT * FROM test_table;')

for row in data:
    print(row['id'])     # access the 'id' column
    print(row['value'])  # access the 'value' column
```

If the query returns no rows, an empty list is returned.

### Parameterized Queries

Both `select` and `update` support parameterized queries using `?` as a placeholder. Placeholders are replaced in order by the values you provide. This is the recommended way to pass user data into queries, as it prevents SQL injection:

``` py linenums="1"
# Selects the row where id = 5
data = sql.select('SELECT * FROM test_table WHERE id = ?;', [5])

# Inserts a new row with id=11, value=1
rows_affected = sql.update('INSERT INTO test_table (id, value) VALUES (?, ?);', [11, 1])
```

Multiple placeholders are filled in order: the first `?` is replaced by the first value, the second `?` by the second, and so on.

## Code Example

The following code connects to and performs basic operations on a remote SQL database. The table `test_table` has columns `id` (integer, auto-increment, not null, unique) and `value` (integer, not null):

``` py linenums="1"
import pyspigot as ps # (1)!

db_manager = ps.database_manager() # (2)!

sql = db_manager.connectSql('localhost', '3306', 'test', 'root', '') # (3)!

data = sql.select('SELECT * FROM test_table ORDER BY value DESC;') # (4)!

for row in data: # (5)!
    print(f"id={row['id']}, value={row['value']}")

rows_affected = sql.update('INSERT INTO test_table (id, value) VALUES (?, ?);', [11, 1]) # (6)!
print(f'Rows affected: {rows_affected}') # (7)!

db_manager.disconnect(sql) # (8)!
```

1. Import PySpigot as `ps` to access the database manager.

2. Get the database manager from `ps` and assign it to `db_manager`.

3. Connect to the database using `connectSql`. The returned `SqlDatabase` object is assigned to `sql` and used for all subsequent interactions with this connection.

4. Select all rows from `test_table`, ordered by `value` descending. The result is a list of dicts, each representing one row.

5. Loop through the rows and print the `id` and `value` column from each row.

6. Insert a new row with `id=11` and `value=1`. The number of rows affected is returned.

7. Print the number of rows affected.

8. Close the connection. Not strictly required on script stop, but good practice when done early.

???+ warning

    The example above runs on the main server thread. Database operations are *I/O operations* — if the connection is slow or the query takes time, this will cause the server to hang. See the async example below.

## Code Example, Asynchronous

The following example wraps the database interactions in asynchronous tasks to avoid blocking the server thread. The initial connection is kept synchronous, because a failed connection should abort script execution before any other logic runs.

The async tasks use the task manager's callback pattern: `runSyncCallbackTask` runs an async function and then calls back to the main server thread with the result.

``` py linenums="1"
import pyspigot as ps

task_manager = ps.task_manager()
db_manager = ps.database_manager()

sql = db_manager.connectSql('localhost', '3306', 'test', 'root', '') # (1)!

def select_data(): # (2)!
    return sql.select('SELECT * FROM test_table ORDER BY value DESC;')

def on_select_complete(data): # (3)!
    for row in data:
        print(f"id={row['id']}, value={row['value']}")

task_manager.runSyncCallbackTask(select_data, on_select_complete) # (4)!

def insert_row(): # (5)!
    rows_affected = sql.update('INSERT INTO test_table (id, value) VALUES (?, ?);', [11, 1])
    print(f'Rows affected: {rows_affected}')

task_manager.runTaskAsync(insert_row) # (6)!
```

1. The connection is established synchronously. If it fails, an exception is thrown here and the script stops before any other code runs.

2. Define the function that performs the SELECT. This will run asynchronously.

3. Define the callback that handles the results. This runs back on the main server thread once the async task finishes, making it safe to interact with the Minecraft API here.

4. Run `select_data` asynchronously, then call `on_select_complete` on the main thread with the returned data.

5. Define the async insert function.

6. Run the insert function asynchronously. Since it only prints to console (not interacting with the Minecraft API), no callback to the main thread is needed.

## Summary

- Use `connectSql(host, port, database, username, password)` for a straightforward connection, or one of the other `connectSql` overloads for more control.
- `connectSql` returns an `SqlDatabase` object used to run queries and updates.
- `select(sql)` returns a **list of rows**. Each row is a dict where keys are column names and values are the column data for that row.
- `update(sql)` returns the number of rows affected.
- Use `?` placeholders in SQL statements and pass corresponding values as a list to prevent SQL injection.
- Database interactions are I/O operations and should be run asynchronously to avoid server lag.
- Call `disconnect(database)` when you are done with a connection.
