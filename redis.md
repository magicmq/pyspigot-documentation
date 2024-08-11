# Redis Manager

PySpigot includes a Redis manager to allow you to connect to and interact with redis servers. Under the hood, PySpigot utilizes [lettuce](https://lettuce.io/) for interfacing with remote Redis server instances. There are two ways the RedisManager allows you to interact with redis servers: by issuing commands (I.E. interacting with the redis database) and by publishing and subscribing to redis' [pub/sub messaging system](https://redis.io/docs/latest/develop/interact/pubsub/). These are implemented in PySpigot via the RedisCommandClient and RedisPubSubClient, respectively. PySpigot also includes a generic ScriptRedisClient that you may create if you would like to work with other aspects of Redis such as Redis Sentinel or Redis Cluster.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the redis manager into your script.

This is not a comprehensive guide to working with redis. Please seek out the appropriate tutorials/information if you're unsure.

## Redis Client Type

For organizational purposes, there are three different types of redis clients available to use based on your specific use case. These are specified using the `ClientType` parameter when initializing/opening a redis client through the Redis Manager. The client types available are:

- `ClientType.BASIC`: This is a generic redis client with no built-in functionality. You can use it if you have a specific use case that the other two redis client types aren't well-suited for.
- `ClientType.COMMAND`: This client type allows for submitting and executing commands on a redis server.
- `ClientType.PUB_SUB`: This client allows for connection to and publishing/subscription to redis pub/sub messaging channels.

Which client you use will depend on your specific use case. For example, if you wish to suscribe to and publish to a redis pub/sub channel, you should use the `PUB_SUB` client type. If you want to do more than one thing at a time, you should open a new connection for each type of activity you wish to do. For example, if you wish to utilize pub/sub messaging *and* commands, you should open two connections, one with the `COMMAND` client type, and another for the `PUB_SUB` client type.

?> The lettuce library includes functionality for both synchronous and asynchronous operations, and this is also reflected in PySpigot. For example, the pub/sub client has ability to publish messages synchronously and asynchronously. In most cases, you should be performing operations asynchronously.

### Generic (Basic) Redis Client

The basic redis client provides access to the underlying RedisClient for you to access whatever you wish. No other functionality is implemented. The object that corresponds to this client type is `ScriptRedisClient`.

Available functions:

- `getRedisURI()`: Get the RedisURI associated with the client.
- `getClientOptions()`: Get the ClientOptions associated with the client.
- `getRedisClient()`: Get the underlying redis client associated with the `ScriptRedisClient`.

For more information, see the [Basic Usage](https://github.com/redis/lettuce/wiki/Basic-usage) section of the lettuce documentation. See the [Code Examples](#code-examples) section below for example usage.

### Command Client

The command client allows for submitting and executing commands on the redis server. Lettuce supports 400+ commands; these can all be viewed in the link to the lettuce documentation at the end of this section. The object that corresponds to this client type is `RedisCommandClient`.

Available functions:

- All functions in the generic `ScriptRedisClient` as outlined above, plus:
- `getConnection()`: Returns the stateful redis connection of the client.
- `getCommands()`: Returns a `RedisCommands` object representing the RedisCommands API for the client connection.
- `getAsyncCommands()`: Returns a `RedisAsyncCommands` object representing the RedisAsyncCommands API for the client connection.

For more information, see the [Command Interfaces](https://github.com/redis/lettuce/wiki/Command-Interfaces-(4.0)) section of the lettuce documentation. See the [Code Examples](#code-examples) section below for example usage.

### Pub/Sub Client

The pub/sub client allows for subscribing and publishing to messaging channels on a redis server. The object that corresponds to this client type is `RedisPubSubClient`.

Available functions:

- All functions in the generic `ScriptRedisClient` as outlined above, plus:
- `registerListener(function, channel)`: Registers a new synchronous listener. Takes a function, which will be called when a message is received, and `channel`, the name of the channel to listen to. Returns a `ScriptPubSubListener` object, which represents the listener that was registered.
- `registerSyncListener(function, channel)`: Registers a new synchronous listener. Takes a function, which will be called when a message is received, and `channel`, the name of the channel to listen to. Returns a `ScriptPubSubListener` object, which represents the listener that was registered.
- `registerAsyncListener(function, channel)`: Registers a new asynchronous listener. Takes a function, which will be called when a message is received, and `channel`, the name of the channel to listen to. Returns a `ScriptPubSubListener` object, which represents the listener that was registered.
- `unregisterListener(listener)`: Unregisters a listener. Takes a `ScriptPubSubListener` to unregister.
- `unregisterListeners(channel)`: Unregisters all listeners listening on the provided channel.
- `publish(channel, message)`: Publishes a message to the given channel synchronously. Takes the channel to publish to, and the message to publish. Returns a number representing the number of clients that received the message.
- `publishSync(channel, message)`: Publishes a message to the given channel synchronously. Takes the channel to publish to, and the message to publish. Returns a number representing the number of clients that received the message.
- `publishAsync(channel, message)`: Publishes a message to the given channel asynchronously. Takes the channel to publish to, and the message to publish. Returns a future that will return a number representing the number of clients that received the message when the operation completes.

For more information, see the [Publish/Subscribe](https://github.com/redis/lettuce/wiki/Pub-Sub) section of the lettuce documentation. See the [Code Examples](#code-examples) section below for example usage.

## Using the Redis Manager

There are several functions available for you to use in the redis manager to facilitate interaction with a redis server. They are:

- `newRedisURI()`: Returns a new, empty RedisURI builder for convenience.
- `newClientOptions()`: Returns a new ClientOptions builder for convenience.
- `openRedisClient(clientType, ip, port, password)`: Opens a connection with the remote redis server with the specified IP, port, and password, using the specified client type. Uses the default client options.
- `openRedisClient(clientType, ip, port, password, clientOptions)`: Opens a connection with the remote redis server with the specified IP, port, and password, using the specified client type. Uses the provided client options.
- `openRedisClient(clientType, redisURI)`: Opens a connection with the remote redis server with the given RedisURI connection string, using the specified client type. Uses the default client options.
- `openRedisClient(clientType, redisURI, clientOptions)`: Opens a connection with the remote redis server with the given RedisURI connection string, using the specified client type. Uses the provided client options.
- `closeRedisClient(client)`: Closes the provided redis client.
- `closeRedisClientAsync(client)`: Closes the provided redis client asynchronously.

### The RedisURI

The RedisURI builder is a convenience object that allows you to easily build a URI connection string for connecting to a remote redis server. Using a URI is probably the most convenient way to establish a connection with a remote redis server, because you can also specify connection settings within the URI, in addition to IP, port, password, etc. For more information on usage, see the [lettuce documentation](https://github.com/redis/lettuce/wiki/Redis-URI-and-connection-details).

A `newRedisURI()` function is provided in the redis manager for convenience in obtaining a new RedisURI builder object.

### The redis ClientOptions

The ClientOptions builder object is a convenience object provided by lettuce that allows you to have greater control over the settings that correspond to the connection. For example, it allows you to set auto reconnect, buffer usage ratio, and request queue size. For more information on ClientOptions, see the [lettuce documentation](https://github.com/redis/lettuce/wiki/Client-Options).

A `newClientOptions()` function is provided in the redis manager for convenience in obtaining a new ClientOptions builder object.

## Code Examples

