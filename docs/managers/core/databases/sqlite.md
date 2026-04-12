# SQLite

PySpigot's database manager supports connecting to SQLite databases, either as a persistent file on disk or as a temporary in-memory database.

For instructions on importing the database manager into your script, visit the [General Information](../../usage.md) page.

???+ info

    This is not a comprehensive guide to SQLite. Please seek out appropriate tutorials and documentation if you are unfamiliar with SQLite.

## Database Manager Usage

The following functions are available on the database manager for connecting to an SQLite database:

- `connectSQLite()`: Connects to an **in-memory** SQLite database. The database exists only in memory and is lost when the connection is closed.
- `connectSQLite(file_path)`: Connects to an SQLite database at the given file path. The path can be relative (relative to the root of the Minecraft/proxy server) or absolute. If no file exists at the given path, a new database file is created automatically.
- `connectSQLite(file_path, create_new_file)`: Same as above, but `create_new_file` controls whether a new file is created if one does not already exist. Pass `True` to create the file automatically (default behavior), or `False` to skip file creation.
- `disconnect(database)`: Disconnects and closes an open SQLite database connection. Accepts the `SQLiteDatabase` object returned by `connectSQLite`.

All `connectSQLite` functions return an `SQLiteDatabase` object, which is the interface used to run statements against the database.

???+ tip

    You do not need to call `disconnect` when your script is stopped or unloaded — PySpigot will automatically close all open connections. However, if you are done with a connection before the script stops, it is good practice to close it manually.

## The SQLiteDatabase Object

When any `connectSQLite` function succeeds, it returns an `SQLiteDatabase` object. This object exposes the following methods:

### Querying and Updating

- `select(sql)`: Executes a SELECT statement and returns the results as a list of dicts. Each element in the list is a dict representing one row, where keys are column names and values are the corresponding column data.
- `select(sql, values...)`: Same as above, with parameterized values that replace `?` placeholders. Pass values as a list, e.g. `[10, 'hello']`.
- `update(sql)`: Executes an INSERT, UPDATE, DELETE, or DDL statement. Returns the number of rows affected.
- `update(sql, values...)`: Same as above, with parameterized values.
- `execute(sql)`: Executes any SQL statement. If the statement is a SELECT or PRAGMA, returns the results as a list of dicts (same format as `select`). For all other statements (INSERT, UPDATE, etc.), executes the update and returns `None`. This method mirrors the behavior of Python's built-in `sqlite3.cursor.execute()`.
- `execute(sql, values...)`: Same as above, with parameterized values.
- `executemany(sql, values)`: Executes a parameterized statement multiple times, once for each set of values provided. `values` is a list of arrays, where each inner array is one set of parameters. This mirrors Python's `sqlite3.cursor.executemany()`.
- `getConnection()`: Returns the underlying JDBC `Connection` object for advanced use.

### Transactions

By default, SQLite auto-commits each statement. You can disable auto-commit to group statements into a transaction:

- `setAutoCommit(auto_commit)`: Enables or disables auto-commit. Pass `False` to begin manual transaction control.
- `getAutoCommit()`: Returns `True` if auto-commit is enabled, `False` if it is disabled.
- `commit()`: Commits the current transaction. Has no effect if auto-commit is enabled.
- `rollback()`: Rolls back the current transaction. Has no effect if auto-commit is enabled.

### Backup and Restore

These methods are particularly useful when working with in-memory databases that need to be persisted to disk (or vice versa):

- `backup(file_name)`: Backs up the current database to the given file path. The path can be relative or absolute. This commits any pending transaction before backing up.
- `restore(file_name)`: Restores the database from the given file path. The path can be relative or absolute. This commits any pending transaction before restoring.

### Select Results

The `select` and `execute` functions (when used with SELECT) return a **list of rows**. Each row is a dict-like object where keys are column names and values are the data in that column for that row:

``` py linenums="1"
data = sqlite.select('SELECT * FROM test_table;')

for row in data:
    print(row['id'])     # access the 'id' column
    print(row['value'])  # access the 'value' column
```

If the query returns no rows, an empty list is returned.

### Parameterized Queries

Both `select`, `update`, and `execute` support `?` placeholders that are replaced in order by the values you provide:

``` py linenums="1"
# Selects the row where id = 5
data = sqlite.select('SELECT * FROM test_table WHERE id = ?;', [5])

# Inserts a new row with id=11, value=1
sqlite.update('INSERT INTO test_table (id, value) VALUES (?, ?);', [11, 1])

# Same insert, using execute
sqlite.execute('INSERT INTO test_table (id, value) VALUES (?, ?);', [11, 1])
```

## Code Example

The following code connects to a file-based SQLite database and performs basic operations. The table `test_table` has columns `id` (integer, auto-increment, not null, unique) and `value` (integer, not null):

``` py linenums="1"
import pyspigot as ps # (1)!

db_manager = ps.database_manager() # (2)!

sqlite = db_manager.connectSQLite('plugins/PySpigot/test_database.db') # (3)!

sqlite.execute('CREATE TABLE IF NOT EXISTS test_table (id INTEGER PRIMARY KEY AUTOINCREMENT, value INTEGER NOT NULL);') # (4)!

sqlite.execute('INSERT INTO test_table (value) VALUES (?);', [42]) # (5)!

data = sqlite.select('SELECT * FROM test_table ORDER BY value DESC;') # (6)!

for row in data: # (7)!
    print(f"id={row['id']}, value={row['value']}")

db_manager.disconnect(sqlite) # (8)!
```

1. Import PySpigot as `ps` to access the database manager.

2. Get the database manager and assign it to `db_manager`.

3. Connect to (or create) an SQLite database at the given path relative to the server root. The returned `SQLiteDatabase` is assigned to `sqlite`.

4. Create the table if it does not already exist, using `execute` which handles both DDL and DML statements.

5. Insert a new row with `value=42`. The `execute` method handles this as an update (non-SELECT) and returns `None`.

6. Select all rows, ordered by `value` descending. The result is a list of dicts, each representing one row.

7. Loop through the rows and print each row's `id` and `value`.

8. Close the connection.

## Code Example, In-Memory Database

The following code creates a temporary in-memory SQLite database, performs operations on it, and then backs it up to a file before closing:

``` py linenums="1"
import pyspigot as ps

db_manager = ps.database_manager()

sqlite = db_manager.connectSQLite() # (1)!

sqlite.execute('CREATE TABLE test_table (id INTEGER PRIMARY KEY AUTOINCREMENT, value INTEGER NOT NULL);')
sqlite.execute('INSERT INTO test_table (value) VALUES (?);', [1])
sqlite.execute('INSERT INTO test_table (value) VALUES (?);', [2])
sqlite.execute('INSERT INTO test_table (value) VALUES (?);', [3])

data = sqlite.select('SELECT * FROM test_table;')
for row in data:
    print(f"id={row['id']}, value={row['value']}")

sqlite.backup('plugins/PySpigot/backup.db') # (2)!

db_manager.disconnect(sqlite) # (3)!
```

1. Connect to a new in-memory database. No file is created; everything exists only in memory.

2. Before closing, back up the in-memory database to a file so the data is not lost.

3. Close the in-memory database connection.

## Code Example, Transactions

The following demonstrates manual transaction control using auto-commit disabled:

``` py linenums="1"
import pyspigot as ps

db_manager = ps.database_manager()

sqlite = db_manager.connectSQLite('plugins/PySpigot/test_database.db')
sqlite.execute('CREATE TABLE IF NOT EXISTS accounts (id INTEGER PRIMARY KEY, balance INTEGER NOT NULL);')

sqlite.setAutoCommit(False) # (1)!

try:
    sqlite.execute('INSERT INTO accounts (id, balance) VALUES (?, ?);', [1, 100])
    sqlite.execute('INSERT INTO accounts (id, balance) VALUES (?, ?);', [2, 200])
    sqlite.commit() # (2)!
except Exception as e:
    sqlite.rollback() # (3)!
    print(f'Transaction failed, rolled back: {e}')

sqlite.setAutoCommit(True)
db_manager.disconnect(sqlite)
```

1. Disable auto-commit to start manual transaction control.

2. If all statements succeed, commit the transaction to persist the changes.

3. If any statement fails, roll back to undo all changes made in this transaction.

## Summary

- Use `connectSQLite(file_path)` to connect to a file-based database, or `connectSQLite()` for an in-memory database.
- `connectSQLite` returns an `SQLiteDatabase` object used to run statements.
- `select(sql)` returns a **list of rows**. Each row is a dict where keys are column names and values are the column data for that row.
- `update(sql)` returns the number of rows affected.
- `execute(sql)` handles any statement type — use it as a single interface for both SELECT and non-SELECT statements.
- `executemany(sql, values)` runs a parameterized statement repeatedly for a list of value sets.
- Use `backup(file_name)` and `restore(file_name)` to persist or load in-memory databases.
- Use `setAutoCommit(False)`, `commit()`, and `rollback()` for transaction control.
- The SQLite JDBC driver must be available at runtime. Many server environments include it, but if not, it must be installed manually.
