# 协议管理器

!> 协议管理器是一个 *可选* 的管理器。只有当 ProtocolLib 插件在 PySpigot 插件启用时存在于服务器上时，才应该访问此管理器。

PySpigot 包含了一个与 ProtocolLib 交互的管理器，如果你想在脚本中处理数据包的话。

有关如何将协议管理器导入到你的脚本中的说明，请参阅 [一般信息](writingscripts#pyspigot-的管理器) 页面。

!> 所有数据包监听器都是从 *异步* 上下文中调用的。换句话说，它们是从主服务器线程之外的线程中调用的。不要在此上下文中调用任何 Bukkit 或 PySpigot API，否则可能会出现问题。

# 协议管理器使用方法

协议管理器提供了几个用于注册和注销数据包监听器的函数：

- `registerPacketListener(function, packet_type)`
- `registerPacketListener(function, packet_type, listener_priority)`：`listener_priority` 是监听器的优先级。这类似于 Bukkit 事件的 EventPriority。
    - 关于监听器优先级的信息，请参见 ProtocolLib 的 [ListenerPriority 类](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/events/ListenerPriority.html)。
- `unregisterPacketListener(packet_listener)`：接受一个由注册函数返回的数据包监听器。
- `createPacket(packet_type)`：创建并返回具有给定类型的包。参见 [PacketType 类](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/PacketType.html)。
- `sendServerPacket(player, packet)`：向提供的玩家发送一个包。
- `broadcastServerPacket(packet)`：向服务器上的所有玩家广播一个包。
- `broadcastServerPacket(packet, entity)`：向在发送包时正在监视给定实体的所有玩家广播一个包。
- `broadcastServerPacket(packet, entity, include_tracker)`：向正在监视给定实体的所有玩家广播一个包。`include_tracker` 用于指定包是否也应广播给实体（除了监视实体的玩家），例如 `true` 或 `false`。
- `broadcastServerPacket(packet, origin, max_observer_distance)`：向从原点位置（中心点）一定最大观察距离内的所有玩家广播一个包。
- `broadcastServerPacket(packet, target_players)`：向目标玩家列表广播一个包。

?> 当你的脚本停止或卸载时，**不需要** 注销你的数据包监听器。PySpigot 会为你处理这个问题。

## 异步监听器

协议管理器还支持注册异步监听器。异步监听器允许你延迟数据包传输等功能。你还可以注册超时监听器来处理在发送过程中超时的数据包。

要获取异步协议管理器，调用 `async()`。例如，

```python
import pyspigot as ps

async_manager = ps.protocol.async()
```

以下函数可用于异步协议管理器：

- `registerAsyncPacketListener(function, packet_type)`
- `registerAsyncPacketListener(function, packet_type, listener_priority)`
- `registerTimeoutPacketListener(function, packet_type)`
- `registerTimeoutPacketListener(function, packet_type, listener_priority)`
- `unregisterAsyncPacketListener(packet_listener)`：接受一个由注册函数返回的数据包监听器。

?> 有关异步和超时监听器的更详细信息，请参阅 ProtocolLib 的文档。

# 代码示例

让我们来看一下下面的代码，该代码定义并注册了一个聊天数据包监听器：

```python
import pyspigot as ps
from com.comphenix.protocol import PacketType

def chat_packet_event(event):
    packet = event.getPacket()
    message = packet.getStrings().read(0)
    print('Player sent a chat! Their message was: ' + event.getMessage())

packet_listener = ps.protocol.registerPacketListener(chat_packet_event, PacketType.Play.Client.CHAT)
```

第 1 行，我们导入 PySpigot 作为 `ps` 以便利用协议管理器 (`protocol`)。

第 2 行，我们从 ProtocolLib 导入 `PacketType`。这将用于定义我们要监听的数据包。

第 4 行，我们定义 `chat_packet_event`，这是一个在拦截到我们监听的数据包时将被调用的函数。这个函数有一个参数 `event`，代表发生的事件（接收到聊天数据包）。

第 4 和 5 行，我们从事件中获取数据包并读取数据包中的数据。

所有数据包监听器都必须通过 PySpigot 的协议管理器进行注册。注册数据包监听器的方式与

- 第一个参数接受一个函数，在事件触发时调用。
- 第二个参数接受要监听的事件。

`registerPacketListener` 函数返回一个 `ScriptPacketListener`，表示已注册的数据包监听器。这可用于稍后注销数据包监听器。

因此，在第 7 行，我们调用协议管理器来注册数据包监听器，传递我们在第 5 行定义的函数 `chat_packet_event` 以及我们要监听的数据包 `PacketType.Play.Client.CHAT`，并将返回的值赋给 `packet_listener`。

## 总结：{docsify-ignore}

- 为了定义你的监听器的数据包类型，你必须从 ProtocolLib 导入 `PacketType`。
- 所有数据包监听器都应该在你的脚本中定义为函数，接受一个参数，即数据包事件（参数名称可以自定义）。
- 所有数据包监听器都必须通过 PySpigot 的数据包管理器进行注册。这可以通过多种方式完成，但最基本的方法是使用 `registerPacketListener(function, packet_type)`。
- 数据包监听器是 *异步* 调用的。任何与 Bukkit 或 PySpigot 交互的代码都应 *同步* 运行。你可以通过使用任务管理器 (`runTask(function)`) 来实现这一点。
- 在注册数据包监听器时，注册函数都会返回一个 `ScriptPacketListener`，这可用于注销监听器。
