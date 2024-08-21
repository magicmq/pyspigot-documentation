# Redis Manager

PySpigot 包含了一个 Redis 管理器，用于让你连接并交互 Redis 服务器。在内部，PySpigot 利用了 [lettuce](https://lettuce.io/) 来与远程 Redis 服务器实例进行交互。RedisManager 允许你通过两种方式与 Redis 服务器交互：一是执行命令（即与 Redis 数据库交互），二是发布和订阅 Redis 的 [发布/订阅消息系统](https://redis.io/docs/latest/develop/interact/pubsub/)。这两种交互方式分别通过 `RedisCommandClient` 和 `RedisPubSubClient` 实现。PySpigot 还包括了一个通用的 `ScriptRedisClient`，如果你想要处理 Redis 的其他方面，例如 Redis Sentinel 或 Redis 集群，你可以创建此类客户端。

请参阅 [一般信息](writingscripts#PySpigot-的管理器) 页面了解如何将 Redis 管理器导入到你的脚本中的说明。

这不是一个全面的 Redis 使用指南。如果你不确定如何操作，请查找适当的教程或信息。

# Redis 客户端类型

为了便于组织，有三种不同类型的 Redis 客户端可供使用，具体取决于你的特定用途。这些类型是在通过 Redis 管理器初始化/打开 Redis 客户端时使用 `ClientType` 参数指定的。可用的客户端类型包括：

- `ClientType.BASIC`: 这是一个通用的 Redis 客户端，没有内置的功能。如果你有一个特定的用途，而其他两种客户端类型不适合，则可以使用它。
- `ClientType.COMMAND`: 此客户端类型允许向 Redis 服务器提交和执行命令。
- `ClientType.PUB_SUB`: 此客户端允许连接到 Redis 发布/订阅消息通道，并在其上发布/订阅消息。

你使用的客户端类型取决于你的特定用途。例如，如果你想订阅和发布到 Redis 的发布/订阅通道，你应该使用 `PUB_SUB` 客户端类型。如果你想同时做多件事，你应该为每种活动打开一个新的连接。例如，如果你想利用发布/订阅消息和命令，你应该打开两个连接，一个使用 `COMMAND` 客户端类型，另一个使用 `PUB_SUB` 客户端类型。

?> lettuce 库包含了同步和异步操作的功能，这同样反映在 PySpigot 中。例如，发布/订阅客户端具有同步和异步发布消息的能力。在大多数情况下，你应该异步执行操作。

## 通用（基本）Redis 客户端

基本 Redis 客户端为你提供了访问底层 RedisClient 的途径，以便你可以访问任何你需要的东西。除此之外没有实现其他功能。与此客户端类型对应的对象是 `ScriptRedisClient`。

可用函数：

- `getRedisURI()`: 获取与客户端关联的 RedisURI。
- `getClientOptions()`: 获取与客户端关联的 ClientOptions。
- `getRedisClient()`: 获取与 `ScriptRedisClient` 关联的底层 Redis 客户端。

更多信息，请参阅 lettuce 文档中的 [基本使用](https://github.com/redis/lettuce/wiki/Basic-usage) 部分。下面的 [代码示例](#代码示例) 部分提供了示例用法。

## 命令客户端

命令客户端允许向 Redis 服务器提交和执行命令。lettuce 支持 400 多个命令；所有这些命令都可以在本节末尾链接的 lettuce 文档中查看。与此客户端类型对应的对象是 `RedisCommandClient`。

可用函数：

- 上面概述的所有通用 `ScriptRedisClient` 的函数，再加上：
- `getConnection()`: 返回客户端的状态化 Redis 连接。
- `getCommands()`: 返回代表客户端连接的 RedisCommands API 的 `RedisCommands` 对象。
- `getAsyncCommands()`: 返回代表客户端连接的 RedisAsyncCommands API 的 `RedisAsyncCommands` 对象。

更多信息，请参阅 lettuce 文档中的 [命令接口](https://github.com/redis/lettuce/wiki/Command-Interfaces-(4.0)) 部分。下面的 [代码示例](#代码示例) 部分提供了示例用法。

## 发布/订阅客户端

发布/订阅客户端允许在 Redis 服务器上的消息通道上订阅和发布消息。与此客户端类型对应的对象是 `RedisPubSubClient`。

可用函数：

- 上面概述的所有通用 `ScriptRedisClient` 的函数，再加上：
- `registerListener(function, channel)`: 注册一个新的同步监听器。接受一个函数，当接收到消息时将调用该函数，以及 `channel`，即要监听的通道名称。返回一个 `ScriptPubSubListener` 对象，表示已注册的监听器。
- `registerSyncListener(function, channel)`: 注册一个新的同步监听器。接受一个函数，当接收到消息时将调用该函数，以及 `channel`，即要监听的通道名称。返回一个 `ScriptPubSubListener` 对象，表示已注册的监听器。
- `registerAsyncListener(function, channel)`: 注册一个新的异步监听器。接受一个函数，当接收到消息时将调用该函数，以及 `channel`，即要监听的通道名称。返回一个 `ScriptPubSubListener` 对象，表示已注册的监听器。
- `unregisterListener(listener)`: 取消注册监听器。接受一个 `ScriptPubSubListener` 来取消注册。
- `unregisterListeners(channel)`: 取消注册所有监听指定通道的监听器。
- `publish(channel, message)`: 同步地向指定通道发布一条消息。接受要发布的通道和消息。返回一个数字，表示接收消息的客户端数量。
- `publishSync(channel, message)`: 同步地向指定通道发布一条消息。接受要发布的通道和消息。返回一个数字，表示接收消息的客户端数量。
- `publishAsync(channel, message)`: 异步地向指定通道发布一条消息。接受要发布的通道和消息。返回一个 Future，当操作完成时返回一个数字，表示接收消息的客户端数量。

更多信息，请参阅 lettuce 文档中的 [发布/订阅](https://github.com/redis/lettuce/wiki/Pub-Sub) 部分。下面的 [代码示例](#代码示例) 部分提供了示例用法。

# 使用 Redis 管理器

Redis 管理器中有几个函数供你使用，以方便与 Redis 服务器进行交互。它们是：

- `newRedisURI()`: 返回一个新的空 RedisURI 构造器，以方便使用。
- `newClientOptions()`: 返回一个新的 ClientOptions 构造器，以方便使用。
- `openRedisClient(clientType, ip, port, password)`: 使用指定的 IP、端口和密码，以及指定的客户端类型，打开与远程 Redis 服务器的连接。使用默认的客户端选项。
- `openRedisClient(clientType, ip, port, password, clientOptions)`: 使用指定的 IP、端口和密码，以及指定的客户端类型，打开与远程 Redis 服务器的连接。使用提供的客户端选项。
- `openRedisClient(clientType, redisURI)`: 使用指定的 RedisURI 连接字符串和指定的客户端类型，打开与远程 Redis 服务器的连接。使用默认的客户端选项。
- `openRedisClient(clientType, redisURI, clientOptions)`: 使用指定的 RedisURI 连接字符串和指定的客户端类型，打开与远程 Redis 服务器的连接。使用提供的客户端选项。
- `closeRedisClient(client)`: 关闭提供的 Redis 客户端。
- `closeRedisClientAsync(client)`: 异步关闭提供的 Redis 客户端。

?> 如果你完成了对 Redis 客户端的使用，遵循良好实践，应该关闭它。如果你在脚本停止或终止时有任何打开的 Redis 客户端，这些打开的客户端将自动关闭。如果在交易期间或等待交易完成时关闭 Redis 客户端，客户端将尝试等待未完成交易的完成，但无法保证交易会成功完成。

## RedisURI

RedisURI 构造器是一个方便的对象，它允许你轻松地构建用于连接到远程 Redis 服务器的 URI 连接字符串。使用 URI 可能是最方便的方式之一来建立与远程 Redis 服务器的连接，因为除了 IP、端口、密码等之外，你还可以在 URI 中指定连接设置。有关用法的更多信息，请参阅 [lettuce 文档](https://github.com/redis/lettuce/wiki/Redis-URI-and-connection-details)。

Redis 管理器中提供了一个 `newRedisURI()` 函数，以便方便地获取新的 RedisURI 构造器对象。

## Redis ClientOptions

ClientOptions 构造器对象是由 lettuce 提供的一个方便的对象，它允许你对与连接相关的设置拥有更大的控制权。例如，它可以让你设置自动重连、缓冲区使用比率和请求队列大小。有关 ClientOptions 的更多信息，请参阅 [lettuce 文档](https://github.com/redis/lettuce/wiki/Client-Options)。

Redis 管理器中提供了一个 `newClientOptions()` 函数，以便方便地获取新的 ClientOptions 构造器对象。

# 代码示例

## 通用客户端示例

以下示例使用基本客户端连接到远程 Redis 服务器。

```python
import pyspigot as ps
from dev.magicmq.pyspigot.manager.redis import ClientType

redis = ps.redis_manager()

basic_client = redis.openRedisClient(ClientType.BASIC, 'localhost', '6379', None)

client = redis_client.getRedisClient()

# Do something with the redis client...
```

第 1 行，我们导入 PySpigot 作为 `ps` 以使用 Redis 管理器。第 2 行，我们导入 `ClientType` 以便稍后使用。

第 4 行，我们从 `ps` 获取数据库管理器并将其设置为 `redis`。

第 6 行，我们使用 `BASIC` 客户端类型、提供的 IP/地址、端口和无密码打开一个新的 Redis 客户端。我们将连接的客户端赋值给 `basic_client`。

第 8 行，我们从 PySpigot 基本客户端获取底层 Redis 客户端，并将其赋值给 `client`。此时，你可以根据需要自由地使用底层 Redis 客户端。

## 命令客户端示例

以下示例使用命令客户端连接到远程 Redis 服务器并提交命令。

```python
import pyspigot as ps
from dev.magicmq.pyspigot.manager.redis import ClientType

redis = ps.redis_manager()

command_client = redis.openRedisClient(ClientType.COMMAND, 'localhost', '6379', None)

commands = command_client.getCommands()

commands.set('test_record', 'Helloredis!')

print(commands.get('test_record'))
```

第 1 行，我们导入 PySpigot 作为 `ps` 以使用 Redis 管理器。第 2 行，我们导入 `ClientType` 以便稍后使用。

第 4 行，我们从 `ps` 获取数据库管理器并将其设置为 `redis`。

第 6 行，我们使用 `COMMAND` 客户端类型、提供的 IP/地址、端口和无密码打开一个新的 Redis 客户端。我们将连接的客户端赋值给 `command_client`。

第 8 行，我们从命令客户端获取 Redis 命令，并将其赋值给 `commands` 变量。

第 10 行，我们提交一个新的命令记录 `test_record`，其值为 `Helloredis!`。

第 12 行，我们通过从 `commands` 获取命令记录并打印其值来验证命令已被提交。

## 发布/订阅客户端示例

以下示例使用发布/订阅客户端连接到远程 Redis 服务器，并订阅和提交消息到其发布/订阅消息系统。

```python
import pyspigot as ps
from dev.magicmq.pyspigot.manager.redis import ClientType

redis = ps.redis_manager()

pub_sub_client = redis.openRedisClient(ClientType.PUB_SUB, 'localhost', '6379', None)

def message_received(channel, message):
    print('Received message on channel \'' + channel + '\': ' + message)

listener = pub_sub_client.registerAsyncListener(message_received, 'test_channel')

num_received = pub_sub_client.publishAsync('test_channel', 'This is a test message!')

pub_sub_client.unregisterListener(listener)
```

第 1 行，我们导入 PySpigot 作为 `ps` 以使用 Redis 管理器。第 2 行，我们导入 `ClientType` 以便稍后使用。

第 4 行，我们从 `ps` 获取数据库管理器并将其设置为 `redis`。

第 6 行，我们使用 `PUB_SUB` 客户端类型、提供的 IP/地址、端口和无密码打开一个新的 Redis 客户端。我们将连接的客户端赋值给 `pub_sub_client`。

第 8 行，我们定义了一个名为 `message_received` 的新函数，它接受两个参数：`channel`（一个字符串）和 `message`（也是一个字符串）。在函数内部，我们打印一条包含 `channel` 和 `message` 的消息。

第 11 行，我们注册一个新的 *异步* 监听器，传递之前定义的函数 `message_received` 以及我们要监听的通道（在这个例子中是 `test_channel`）。我们将注册的监听器赋值给 `listener`，以便稍后取消注册。

?> 当在 `test_channel` 通道上收到消息时，`message_received` 函数会被自动调用，并将通道名称（`channel` 参数）传递给它，在这种情况下将是 `test_channel`，以及消息的内容（`message` 参数）。

第 13 行，我们异步地向通道 `test_channel` 发布一条内容为 `This is a test message!` 的消息。所有向通道发布消息的函数（无论是同步还是异步）都会返回一个值，表示接收消息的客户端数量。我们将这个值赋值给 `num_received`。

?> 注意由于消息是异步发布的，`num_received` 是一个 [RedisFuture](https://lettuce.io/lettuce-4/release/api/com/lambdaworks/redis/RedisFuture.html) 对象。需要额外的代码（这里没有详细说明）来从该对象获取值。如果你需要帮助，可以在 Discord 上提问。

第 15 行，我们通过将 `listener` 传递给 `unregisterListener` 函数来取消注册之前注册的监听器。

!> 如前所述，包括了同步和异步执行 Redis 操作的函数。一般来说，最好异步执行操作，以避免服务器挂起和延迟（因为大多数这些函数涉及与远程 Redis 服务器的交互，这些都是 I/O 操作，因此相对缓慢）。

## 总结

- RedisManager 允许你连接到并与远程 Redis 服务器交互。
- 有三种可用的客户端类型：`ClientType.BASIC`、`ClientType.COMMAND` 和 `ClientType.PUB_SUB`。你应该使用哪一种取决于你的特定用途。
- 使用 `openRedisClient` 函数（加上客户端类型和其他根据你的具体情况提供的选项）来连接到 Redis 服务器。
- 当连接到 Redis 服务器时，`openRedisClient` 函数会返回一个 Redis 客户端对象（将是一个 `ScriptRedisClient`、`RedisCommandClient` 或 `RedisPubSubClient`，具体取决于指定的客户端类型），然后使用它来与 Redis 交互。
- 与 Redis 交互是一种 *I/O 操作*。除非在非常有限的情况下，否则应使用异步函数而不是同步函数。例如，如果使用 `RedisPubSubClient`，应使用 `publishAsync` 而不是 `publish` 或 `publishSync`。
- Redis 客户端会在脚本停止时自动关闭。在其他任何时候，如果你完成了对 Redis 客户端的使用，应该通过从 Redis 管理器调用 `closeRedisClient` 或 `closeRedisClientAsync` 来关闭它。`closeRedisClient` / `closeRedisClientAsync` 函数接受在打开客户端时返回的 Redis 客户端对象。
