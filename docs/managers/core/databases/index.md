# Database Manager

PySpigot includes a database manager that allows scripts to connect to and interact with several types of databases. Under the hood, PySpigot uses [HikariCP](https://github.com/brettwooldridge/HikariCP) for SQL-type databases and the [MongoDB Java Driver](https://www.mongodb.com/docs/drivers/java-drivers/) for MongoDB.

For instructions on importing the database manager into your script, visit the [General Information](../../usage.md) page.

???+ info

    This is not a comprehensive guide to working with SQL or MongoDB. Please seek out appropriate tutorials and documentation if you are unfamiliar with these database systems.

## Supported Database Types

Although support for each database type is documented on its own page, they are all accessed through the **same database manager** (`database_manager()`). The specific object returned by each connect function determines which database you are working with.

- [MySQL](mysql.md) — Connect to MySQL, MariaDB, PostgreSQL, and other SQL-compatible servers via HikariCP.
- [SQLite](sqlite.md) — Connect to a local SQLite database file, or an in-memory SQLite database.
- [MongoDB](mongodb.md) — Connect to a MongoDB server via the official MongoDB Java Driver.

## JDBC Drivers

Note that in order to interface with an external database framework, Java typically requires a [JDBC](https://en.wikipedia.org/wiki/Java_Database_Connectivity) driver built for that specific database to be present at runtime. On Spigot, this is not an issue, as Spigot bundles JDBC drivers for both MySQL and SQLite (so they're already included). However, on Velocity and BungeeCord, these drivers are *not* bundled, so plugin developers are required to include them in their plugins to use MySQL/SQLite.

Therefore, prior to PySpigot version 0.10.0, users were required to manually download JDBC drivers for MySQL and SQLite on the Velocity and BungeeCord platforms in order to use the MySQL and SQLite managers.

???+ note

    PySpigot has always included the MongoDB JDBC driver in the distributable JAR file for all platforms, so the lack of a MongoDB JDBC driver bundled into the server platform was never an issue.

From PySpigot version 0.10.0 onwards, PySpigot leverages its [internal runtime dependency management system](../../../pyspigot/dependencies.md) to automatically download and make available JDBC drivers for MySQL and SQLite at runtime, so no extra work is needed to interact with these databases, regardless of platform.

Here are the JDBC drivers PySpigot uses for each database (all of these are downloaded and added to the classpath at runtime by the runtime dependency management system):

- MySQL: [mysql-connector-j](https://github.com/mysql/mysql-connector-j)
- SQLite: [sqlite-jdbc](https://github.com/xerial/sqlite-jdbc)
- MongoDB: [mongodb-driver-sync](https://www.mongodb.com/docs/drivers/java/sync/current/)

???+ warning

    As discussed above, PySpigot *does not* download MySQL/SQLite JDBC drivers if you're on a Bukkit-esque platform, because they're assumed to be bundled into the server already. If you're running PySpigot on a fork of Spigot/Paper which, for some reason, does not bundle these drivers, you'll see the following errors in the console:

    - `SQL JDBC driver not found on the class path`
    - `SQLite JDBC driver not found on the class path`

    If you see either of these, you'll need to download the JDBC driver manually and include it as an external library. See the [External Libraries](../../../scripts/externallibraries.md) page for more information.

## Disconnecting

All three database types share the same disconnect function on the database manager:

- `disconnect(database)`: Disconnects and closes the given database connection. Accepts the database object returned by any of the connect functions (`SqlDatabase`, `SQLiteDatabase`, or `MongoDatabase`). Returns `True` if the disconnect was successful.

???+ tip

    Open database connections are closed automatically when your script is stopped or unloaded. If you are finished using a database connection before your script stops, it is good practice to close it manually by calling `disconnect`.

## I/O and Asynchronous Usage

Interacting with any database is an *I/O operation*. If you call database operations on the main server thread, the server **will hang** until each operation completes. For remote databases over a network, this can cause significant server lag. It is strongly recommended to perform all database interactions inside an asynchronous task. See the code examples on each individual page for an async example.
