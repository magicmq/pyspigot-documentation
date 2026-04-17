# Redis Manager

PySpigot includes a Redis manager to allow you to connect to and interact with redis servers. Under the hood, PySpigot utilizes [lettuce](https://lettuce.io/) for interfacing with remote Redis server instances. There are two ways the RedisManager allows you to interact with redis servers: by issuing commands (I.E. interacting with the redis database) and by publishing and subscribing to redis' [pub/sub messaging system](https://redis.io/docs/latest/develop/interact/pubsub/). These are implemented in PySpigot via the RedisCommandClient and RedisPubSubClient, respectively. PySpigot also includes a generic ScriptRedisClient that you may create if you would like to work with other aspects of Redis such as Redis Sentinel or Redis Cluster.

For instructions on importing the redis manager into your script, visit the [General Information](../usage.md) page.

???+ note

    This is not a comprehensive guide to working with redis. Please seek out the appropriate tutorials/information if you're unsure.

## Redis Client Type

For organizational purposes, there are three different types of redis clients available to use based on your specific use case. These are specified using the `ClientType` parameter when initializing/opening a redis client through the Redis Manager. The client types available are:

- `ClientType.BASIC`: This is a generic redis client with no built-in functionality. You can use it if you have a specific use case that the other two redis client types aren't well-suited for.
- `ClientType.COMMAND`: This client type allows for submitting and executing commands on a redis server.
- `ClientType.PUB_SUB`: This client allows for connection to and publishing/subscription to redis pub/sub messaging channels.

Which client you use will depend on your specific use case. For example, if you wish to suscribe to and publish to a redis pub/sub channel, you should use the `PUB_SUB` client type. If you want to do more than one thing at a time, you should open a new connection for each type of activity you wish to do. For example, if you wish to utilize pub/sub messaging *and* commands, you should open two connections, one with the `COMMAND` client type, and another for the `PUB_SUB` client type.

???+ note

    The lettuce library includes functionality for both synchronous and asynchronous operations, and this is also reflected in PySpigot. For example, the pub/sub client has ability to publish messages synchronously and asynchronously. In most cases, you should be performing operations asynchronously.

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

#### Pub/Sub Listener Decorators

PySpigot ships with a `decorators/redis.py` helper module that provides Python **decorators** for registering pub/sub listeners. Using the decorators is the recommended way to register listeners on a `RedisPubSubClient`.

**Importing:**

``` py
from decorators.redis import pub_sub_listener        # (1)!
from decorators.redis import async_pub_sub_listener
```

1. `pub_sub_listener` is an alias for `sync_pub_sub_listener`. Both are available — they are identical.

Apply the decorator to a function to register it as a listener on a channel. The decorator takes the **already-opened** `RedisPubSubClient` as its first argument, and the channel name as its second:

``` py linenums="1"
from decorators.redis import pub_sub_listener, async_pub_sub_listener
from dev.magicmq.pyspigot.manager.redis import ClientType
import pyspigot as ps

redis = ps.redis_manager()
pub_sub_client = redis.openRedisClient(ClientType.PUB_SUB, 'localhost', 6379, None)

@pub_sub_listener(pub_sub_client, 'test_channel') # (1)!
def on_message_sync(channel, message): # (2)!
    print(f'Received synchronously on \'{channel}\': {message}')

@async_pub_sub_listener(pub_sub_client, 'test_channel') # (3)!
def on_message_async(channel, message):
    print(f'Received asynchronously on \'{channel}\': {message}')
```

1. Registers `on_message_sync` as a **synchronous** listener on `test_channel`. The `RedisPubSubClient` must already be open before the decorator is applied.
2. Listener functions receive two arguments: `channel` (the name of the channel the message arrived on) and `message` (the message content).
3. Registers `on_message_async` as an **asynchronous** listener on the same channel.

When decorated, each function gains two attributes:

- `.registered_listener` — the `ScriptPubSubListener` object representing the registered listener.
- `.unregister()` — a convenience method that unregisters the listener.

``` py linenums="1"
on_message_sync.unregister()  # (1)!
```

1. Unregisters the `on_message_sync` listener. After this call, the function will no longer be called when a message arrives on `test_channel`.

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

???+ tip

    If you're finished using a redis client, it is good practice to close it. If you have any open redis clients when your script is stopped or terminated, then these open clients will be closed automatically. If a redis client is closed either during or pending execution of a transaction, the client will attempt to wait for completion of the pending transactions prior to closing, but there is no guarantee that the transactions will complete successfully.

### The RedisURI

The RedisURI builder is a convenience object that allows you to easily build a URI connection string for connecting to a remote redis server. Using a URI is probably the most convenient way to establish a connection with a remote redis server, because you can also specify connection settings within the URI, in addition to IP, port, password, etc. For more information on usage, see the [lettuce documentation](https://github.com/redis/lettuce/wiki/Redis-URI-and-connection-details).

A `newRedisURI()` function is provided in the redis manager for convenience in obtaining a new RedisURI builder object.

### The redis ClientOptions

The ClientOptions builder object is a convenience object provided by lettuce that allows you to have greater control over the settings that correspond to the connection. For example, it allows you to set auto reconnect, buffer usage ratio, and request queue size. For more information on ClientOptions, see the [lettuce documentation](https://github.com/redis/lettuce/wiki/Client-Options).

A `newClientOptions()` function is provided in the redis manager for convenience in obtaining a new ClientOptions builder object.

## Code Examples

### General Client Example

The following example utilizes the basic client to connect to a remote redis server.

``` py linenums="1"
import pyspigot as ps # (1)!
from dev.magicmq.pyspigot.manager.redis import ClientType # (2)!

redis = ps.redis_manager() # (3)!

basic_client = redis.openRedisClient(ClientType.BASIC, 'localhost', '6379', None) # (4)!

client = redis_client.getRedisClient() # (5)!

# Do something with the redis client...
```

1. Here, we import PySpigot as `ps` to utilize the redis manager.

2. Here, we import `ClientType` so it can be used later.

3. Here, we get the database manager from `ps` and set it to `redis`.

4. Here, we open a new redis client with the `BASIC` client type, using the provided IP/address, port, and no password. We assign the connected client to `basic_client`.

5. Here, we fetch the underlying redis client from the PySpigot basic client, and assign it to `client`. At this point, you are able to work with the underlying redis client however you wish.

### Command Client Example

The following example utilizes the command client to connect to and submit a command to a remote redis server.

``` py linenums="1"
import pyspigot as ps # (1)!
from dev.magicmq.pyspigot.manager.redis import ClientType # (2)!

redis = ps.redis_manager() # (3)!

command_client = redis.openRedisClient(ClientType.COMMAND, 'localhost', '6379', None) # (4)!

commands = command_client.getCommands() # (5)!

commands.set('test_record', 'Helloredis!') # (6)!

print(commands.get('test_record')) # (7)!
```

1. Here, we import PySpigot as `ps` to utilize the redis manager.

2. Here, we import `ClientType` so it can be used later.

3. Here, we get the database manager from `ps` and set it to `redis`.

4. Here, we open a new redis client with the `COMMAND` client type, using the provided IP/address, port, and no password. We assign the connected client to `command_client`.

5. Here, we fetch redis commands from the command client and assign it to the `commands` variable.

6. Here, we submit a new command record `test_record` with the value `Helloredis!`.

7. Here, we verify that the command was submitted by getting the command record from `commands` and printing its value.

### Pub/Sub Client Example

The following example utilizes the pub/sub client to connect to a remote redis server and subscribe to and publish messages on its pub/sub messaging system.

``` py linenums="1"
import pyspigot as ps # (1)!
from dev.magicmq.pyspigot.manager.redis import ClientType # (2)!
from decorators.redis import async_pub_sub_listener # (3)!

redis = ps.redis_manager() # (4)!

pub_sub_client = redis.openRedisClient(ClientType.PUB_SUB, 'localhost', 6379, None) # (5)!

@async_pub_sub_listener(pub_sub_client, 'test_channel') # (6)!
def message_received(channel, message): # (7)!
    print(f'Received message on channel \'{channel}\': {message}')

num_received = pub_sub_client.publishAsync('test_channel', 'This is a test message!') # (8)!

message_received.unregister() # (9)!
```

1. Here, we import PySpigot as `ps` to utilize the redis manager.

2. Here, we import `ClientType` so it can be used later.

3. Here, we import the `async_pub_sub_listener` decorator.

4. Here, we get the redis manager from `ps` and assign it to `redis`.

5. Here, we open a new redis client with the `PUB_SUB` client type, connecting to `localhost` on port `6379` with no password. We assign the connected client to `pub_sub_client`.

6. Here, we register `message_received` as an asynchronous listener on `test_channel` using the decorator. The `pub_sub_client` must be open before the decorator is applied.

7. The listener function receives two arguments: `channel` (the name of the channel the message arrived on) and `message` (the message content).

8. Here, we publish a message asynchronously to `test_channel`. The return value is a [RedisFuture](https://lettuce.io/lettuce-4/release/api/com/lambdaworks/redis/RedisFuture.html) that resolves to the number of clients that received the message.

9. Here, we unregister the listener using the `.unregister()` method attached by the decorator.

???+ note

    When a message is received on the `test_channel` channel, the `message_received` function is called automatically, and it will be passed the name of the channel (the `channel` argument), which would be `test_channel` in this case, as well as the content of the message (the `message` argument).

???+ note

    Because the message is published *asynchronously*, `num_received` is a [RedisFuture](https://lettuce.io/lettuce-4/release/api/com/lambdaworks/redis/RedisFuture.html) object. Additional code, not shown here, is required to fetch the value from this object. If you need help with this, ask on Discord.

???+ warning

    As outlined previously, functions are included to execute redis operations both synchronously and asynchronously. In general, it is best to do things asynchronously, to avoid server hangs and lag (since most of these functions perform interactions with a remote redis server, these are all I/O operations and are thus relatively slow to complete).

## Summary

- The RedisManager allows you to connect to and interact with remote redis servers.
- There are three available client types: `ClientType.BASIC`, `ClientType.COMMAND`, and `ClientType.PUB_SUB`. Which one you should use depends on your specific use case.
- Use the `openRedisClient` functions (along with the client type and other provided options, based on your specific situation) to connect to a redis server.
- When connecting to a redis server, a redis client object (will be either a `ScriptRedisClient`, `RedisCommandClient`, or a `RedisPubSubClient`, depending on the specified client type) is returned by the `openRedisClient` functions, which is then used to interact with redis.
- For `RedisPubSubClient` listeners, the recommended approach is to use the `@pub_sub_listener` or `@async_pub_sub_listener` decorators from `decorators/redis.py`. Both decorators require the already-opened `RedisPubSubClient` as their first argument.
- Decorated listener functions gain a `.registered_listener` attribute (the `ScriptPubSubListener`) and an `.unregister()` method.
- Interacting with redis is an *I/O operation*. Except in very limited contexts, asynchronous functions should be used over synchronous ones. For example, if using the `RedisPubSubClient`, `publishAsync` should be used instead of `publish` or `publishSync`, and `@async_pub_sub_listener` should be preferred over `@pub_sub_listener`.
- Redis clients are closed automatically when a script is stopped. At any other time, if you are finished using a redis client, it should be closed by calling either `closeRedisClient` or `closeRedisClientAsync` from the redis manager. The `closeRedisClient`/`closeRedisClientAsync` functions take the redis client object that was returned when opening the client.