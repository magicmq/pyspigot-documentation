# Database Manager

PySpigot includes a database manager that allows you to connect to and interact with SQL database types (including MySQL, MariaDB, PostgreSQL, and more) as well as MongoDB. Under the hood, PySpigot utilizes [HikariCP](https://github.com/brettwooldridge/HikariCP) for interfacing with SQL-type database servers, and the [MongoDB Java Driver](https://www.mongodb.com/docs/drivers/java-drivers/) for interfacing with MongoDB database servers.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the database manager into your script.

This is not a comprehensive guide to working with SQL or MongoDB. Please seek out the appropriate tutorials/information if you're unsure how to use SQL or MongoDB.

## Using the Database Manager