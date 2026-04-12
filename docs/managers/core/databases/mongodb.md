# MongoDB

PySpigot's database manager supports connecting to MongoDB servers via the official [MongoDB Java Driver](https://www.mongodb.com/docs/drivers/java-drivers/).

For instructions on importing the database manager into your script, visit the [General Information](../../usage.md) page.

???+ info

    This is not a comprehensive guide to MongoDB. Please seek out appropriate tutorials and documentation if you are unfamiliar with MongoDB. The [MongoDB Java Driver documentation](https://www.mongodb.com/docs/drivers/java/sync/current/) is also a useful reference for the underlying library that PySpigot uses.

## Database Manager Usage

The following functions are available on the database manager for connecting to a MongoDB server:

- `newMongoClientSettings()`: Returns a new `MongoClientSettings.Builder` for customizing connection settings before connecting.
- `connectMongo(host, port, username, password)`: Connects to a MongoDB server using the provided host, port, and credentials. Pass `None` for `username` and `password` to connect without authentication.
- `connectMongo(host, port, username, password, clientSettings)`: Connects using the provided credentials and a custom `MongoClientSettings`.
- `connectMongo(uri)`: Connects using a [MongoDB connection string URI](https://www.mongodb.com/docs/manual/reference/connection-string/), which allows fine-grained control over connection parameters and options.
- `connectMongo(uri, clientSettings)`: Connects using a connection string URI and a custom `MongoClientSettings`.
- `connectMongo(clientSettings)`: Connects using only a `MongoClientSettings` object. The settings must encode all connection information (e.g., via an `applyConnectionString` call).
- `disconnect(database)`: Disconnects and closes an open MongoDB connection. Accepts the `MongoDatabase` object returned by any `connectMongo` function.

All `connectMongo` functions return a `MongoDatabase` object, which is the interface used to interact with the MongoDB server.

???+ tip

    You do not need to call `disconnect` when your script is stopped or unloaded — PySpigot will automatically close all open connections. However, if you are done with a connection before the script stops, it is good practice to close it manually.

## The MongoClientSettings

The `MongoClientSettings` object gives you finer control over the MongoDB connection — including connection pool settings, read/write preferences, encryption, and more. For full details, see the [MongoDB documentation](https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/connection/mongoclientsettings/).

Use `newMongoClientSettings()` to get a builder, configure it as needed, then call `build()` to produce the final settings object:

``` py linenums="1"
import pyspigot as ps
from com.mongodb import ReadPreference

db_manager = ps.database_manager()

settings_builder = db_manager.newMongoClientSettings()
settings_builder.readPreference(ReadPreference.secondaryPreferred())
settings = settings_builder.build()

mongo = db_manager.connectMongo('localhost', '27017', None, None, settings)
```

???+ warning

    You **must** call `build()` on the settings builder before passing it to a `connectMongo` function.

## The MongoDatabase Object

When any `connectMongo` function succeeds, it returns a `MongoDatabase` object. This object exposes methods organized into the following categories.

### Client

- `getMongoClient()`: Returns the underlying `MongoClient` object for advanced use.

### Helper Factories

These methods create BSON objects and documents that are used as inputs to other methods:

- `createDocument()`: Creates and returns an empty BSON `Document`.
- `createDocument(json)`: Creates a `Document` from the provided JSON string.
- `createDocument(key, value)`: Creates a `Document` with a single key-value pair.
- `createObject()`: Creates and returns an empty `BasicDBObject`.
- `createObject(json)`: Creates a `BasicDBObject` from the provided JSON string.
- `createObject(key, value)`: Creates a `BasicDBObject` with a single key-value pair.
- `fetchNewUpdateOptions()`: Returns a new `UpdateOptions` object, used to configure update operations (e.g., enabling upsert).
- `fetchNewFindOneAndUpdateOptions()`: Returns a new `FindOneAndUpdateOptions` object, used to configure find-and-update operations.

### Database Operations

- `getDatabase(database)`: Returns the underlying `MongoDatabase` object for the given database name. Useful for directly accessing MongoDB's Java API.
- `getDatabaseNames()`: Returns an iterable of all database names on the server.
- `getDatabases()`: Returns an iterable of all databases on the server as BSON `Document` objects.
- `doesDatabaseExist(database)`: Returns `True` if a database with the given name exists on the server.

### Collection Operations

- `createCollection(database, collection)`: Creates a new collection in the specified database. Returns `True` if created, `False` if the collection already exists.
- `deleteCollection(database, collection)`: Drops the specified collection from the database. Returns `True` if deleted, `False` if it did not exist.
- `getCollection(database, collection)`: Returns the `MongoCollection` for the given database and collection name.
- `getCollectionNames(database)`: Returns an iterable of all collection names within the given database.
- `getCollections(database)`: Returns an iterable of all collections within the given database as BSON `Document` objects.
- `doesCollectionExist(database, collection)`: Returns `True` if the specified collection exists in the given database.
- `createCollectionIndex(database, collection, keys)`: Creates an index on the specified collection using the given `Bson` keys document. Returns the name of the created index.

### Getting Documents

- `getDocument(database, collection, filter)`: Returns the first `Document` in the collection that matches the given filter.
- `getDocument(database, collection, filter, projections, sorts)`: Returns the first matching document, with the given field projections and sort criteria applied.
- `getDocuments(database, collection)`: Returns a `FindIterable` of all documents in the collection.
- `getDocuments(database, collection, filter)`: Returns a `FindIterable` of all documents matching the given filter.

### Inserting Documents

- `insertDocument(database, collection, document)`: Inserts a single `Document` into the collection. Returns an `InsertOneResult`.
- `insertDocuments(database, collection, documents)`: Inserts a list of `Document` objects into the collection. Returns an `InsertManyResult`.

### Updating Documents

- `updateDocument(database, collection, filter, update)`: Updates the first document matching the filter using the given `Bson` update. Returns an `UpdateResult`.
- `updateDocument(database, collection, filter, update, updateOptions)`: Same as above, with custom `UpdateOptions` (e.g., to enable upsert).
- `updateDocument(database, collection, filter, updates)`: Updates the first matching document using a list of `Bson` update pipeline stages. Returns an `UpdateResult`.
- `updateDocument(database, collection, filter, updates, updateOptions)`: Same as above, with custom `UpdateOptions`.
- `updateDocuments(database, collection, filter, update)`: Updates **all** documents matching the filter using the given `Bson` update. Returns an `UpdateResult`.
- `updateDocuments(database, collection, filter, update, updateOptions)`: Same as above, with custom `UpdateOptions`.
- `updateDocuments(database, collection, filter, updates)`: Updates all matching documents using a list of `Bson` update pipeline stages. Returns an `UpdateResult`.
- `updateDocuments(database, collection, filter, updates, updateOptions)`: Same as above, with custom `UpdateOptions`.
- `findAndUpdateDocument(database, collection, filter, update)`: Updates the first matching document and returns the **updated document** (after the update is applied).
- `findAndUpdateDocument(database, collection, filter, update, updateOptions)`: Same as above, with custom `FindOneAndUpdateOptions`.
- `findAndUpdateDocument(database, collection, filter, updates)`: Same, using a list of update pipeline stages.
- `findAndUpdateDocument(database, collection, filter, updates, updateOptions)`: Same, with custom `FindOneAndUpdateOptions`.

### Deleting Documents

- `deleteDocument(database, collection, filter)`: Deletes the **first** document matching the filter. Returns a `DeleteResult`.
- `deleteDocuments(database, collection, filter)`: Deletes **all** documents matching the filter. Returns a `DeleteResult`.

## Filters and Updates

MongoDB operations such as `getDocument`, `updateDocument`, and `deleteDocument` take `Bson` filter and update objects. The most common way to create these in scripts is to use the `Filters` and `Updates` builder classes from the MongoDB Java Driver:

``` py linenums="1"
from com.mongodb.client.model import Filters
from com.mongodb.client.model import Updates
from com.mongodb.client.model import Sorts

# A filter that matches documents where 'username' equals 'Alice'
filter = Filters.eq('username', 'Alice')

# A filter that matches documents where 'score' is greater than 50
filter = Filters.gt('score', 50)

# Combine multiple conditions with and/or
filter = Filters.and_(Filters.eq('active', True), Filters.gt('score', 50))

# An update that sets the 'score' field to 100
update = Updates.set('score', 100)

# An update that increments 'score' by 5
update = Updates.inc('score', 5)

# Sort by 'score' descending
sort = Sorts.descending('score')
```

For a complete list of available filters and update operators, see the [MongoDB Java Driver documentation](https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/builders/).

## Code Example

The following code connects to a MongoDB server, creates a collection, inserts documents, queries them, updates one, and deletes one:

``` py linenums="1"
import pyspigot as ps # (1)!
from com.mongodb.client.model import Filters # (2)!
from com.mongodb.client.model import Updates # (3)!

db_manager = ps.database_manager() # (4)!

mongo = db_manager.connectMongo('localhost', '27017', None, None) # (5)!

if not mongo.doesCollectionExist('my_db', 'players'): # (6)!
    mongo.createCollection('my_db', 'players')

mongo.insertDocument('my_db', 'players', mongo.createDocument('username', 'Alice').append('score', 50)) # (7)!
mongo.insertDocument('my_db', 'players', mongo.createDocument('username', 'Bob').append('score', 75))

documents = mongo.getDocuments('my_db', 'players') # (8)!
for doc in documents:
    print(doc)

alice = mongo.getDocument('my_db', 'players', Filters.eq('username', 'Alice')) # (9)!
print(f"Alice's score: {alice.get('score')}")

mongo.updateDocument('my_db', 'players', Filters.eq('username', 'Alice'), Updates.set('score', 100)) # (10)!

mongo.deleteDocument('my_db', 'players', Filters.eq('username', 'Bob')) # (11)!

db_manager.disconnect(mongo) # (12)!
```

1. Import PySpigot as `ps` to access the database manager.

2. Import `Filters` from the MongoDB Java Driver to build query filters.

3. Import `Updates` from the MongoDB Java Driver to build update operations.

4. Get the database manager and assign it to `db_manager`.

5. Connect to the MongoDB server running on localhost. `None` is passed for username and password since authentication is not being used here.

6. Create the `players` collection in `my_db` if it does not already exist.

7. Insert two player documents into the collection. Note that `createDocument` returns a `Document` object, and `.append(key, value)` adds additional fields to it.

8. Retrieve all documents from the `players` collection and print each one.

9. Retrieve the single document where `username` equals `'Alice'`, using a `Filters.eq` filter.

10. Update Alice's `score` field to `100` using `Updates.set`.

11. Delete Bob's document from the collection.

12. Close the connection.

???+ warning

    The above code runs synchronously. MongoDB operations involve network I/O and should be run asynchronously to avoid server lag. Wrap database calls in `task_manager.runTaskAsync(...)` or `task_manager.runSyncCallbackTask(...)` as appropriate.

## Summary

- Use `connectMongo(host, port, username, password)` for a straightforward connection, or one of the other `connectMongo` overloads for more control.
- Pass `None` for `username` and `password` to connect without authentication.
- `connectMongo` returns a `MongoDatabase` object used to interact with the server.
- Use `createDocument` or `createObject` to build BSON documents to insert or use as filters.
- Use `Filters` and `Updates` from `com.mongodb.client.model` to build query filters and update operations.
- `updateDocument` updates only the **first** matching document; `updateDocuments` updates **all** matching documents.
- `findAndUpdateDocument` updates the first matching document and returns the updated document.
- `deleteDocument` deletes the first match; `deleteDocuments` deletes all matches.
- Database interactions are I/O operations and should be run asynchronously to avoid server lag.
- Call `disconnect(database)` when you are done with a connection.
