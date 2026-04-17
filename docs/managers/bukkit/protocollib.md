# Working with ProtocolLib

???+ warning

    The Protocol manager is an *optional* manager. This manager should only be accessed if the ProtocolLib plugin is present on the server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with ProtocolLib if you would like to work with packets in your script.

For instructions on importing the protocol manager into your script, visit the [General Information](../usage.md) page.

## Protocol Decorators

PySpigot ships with a `decorators/protocol.py` helper module that provides Python **decorators** for registering packet listeners. Using the decorators is the recommended way to register packet listeners, as it is cleaner and more Pythonic than calling the protocol manager directly.

???+ warning

    The `decorators/protocol` module checks whether ProtocolLib is available **at import time**. Importing from this module on a server where ProtocolLib is not installed will immediately raise a `ScriptRuntimeException`. Only import this module if you know ProtocolLib is present.

### Importing

``` py
from decorators.protocol import packet_listener
from decorators.protocol import async_packet_listener   # (1)!
from decorators.protocol import timeout_packet_listener # (2)!
```

1. Only needed for [asynchronous packet listeners](#async_packet_listener).
2. Only needed for [timeout packet listeners](#timeout_packet_listener).

### `@packet_listener`

Registers a **synchronous** packet listener. The decorated function is called when the specified packet type is sent or received.

``` py linenums="1"
from decorators.protocol import packet_listener
from com.comphenix.protocol import PacketType # (1)!

@packet_listener(PacketType.Play.Client.CHAT) # (2)!
def chat_packet_event(event): # (3)!
    packet = event.getPacket()
    message = packet.getStrings().read(0)
    print(f'Player sent a chat! Their message was: {message}')
```

1. The `PacketType` class must be imported before it is passed to the decorator.

2. Applying `@packet_listener(PacketType.Play.Client.CHAT)` registers `chat_packet_event` as a synchronous listener for the `CHAT` packet. No separate call to the protocol manager is needed.

3. The function receives the packet event as its only parameter.

???+ tip

    Packet listeners are called **asynchronously** by ProtocolLib. Any code that interacts with the Bukkit API must be run synchronously. Use the task manager's `runTask(function)` to bring work back to the main thread.

An optional `priority` parameter controls the order in which multiple listeners for the same packet type are invoked. It accepts a ProtocolLib `ListenerPriority` and defaults to `ListenerPriority.NORMAL`:

``` py
from decorators.protocol import packet_listener
from com.comphenix.protocol import PacketType
from com.comphenix.protocol.events import ListenerPriority

@packet_listener(PacketType.Play.Client.CHAT, priority=ListenerPriority.HIGH)
def chat_packet_event(event):
    ...
```

For a list of available priorities, see ProtocolLib's [ListenerPriority class](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/events/ListenerPriority.html).

### `@async_packet_listener`

Registers an **asynchronous** packet listener via ProtocolLib's asynchronous manager. Asynchronous listeners allow you to delay packet transmission, among other things.

``` py linenums="1"
from decorators.protocol import async_packet_listener
from com.comphenix.protocol import PacketType

@async_packet_listener(PacketType.Play.Client.CHAT) # (1)!
def async_chat_packet_event(event):
    # Packet transmission can be delayed here...
    pass
```

1. Registers `async_chat_packet_event` as an asynchronous listener. An optional `priority` parameter is also available, defaulting to `ListenerPriority.NORMAL`.

See ProtocolLib's documentation for detailed information on asynchronous listeners.

### `@timeout_packet_listener`

Registers a **timeout** packet listener via ProtocolLib's asynchronous manager. Timeout listeners handle packets that time out during asynchronous processing.

``` py linenums="1"
from decorators.protocol import timeout_packet_listener
from com.comphenix.protocol import PacketType

@timeout_packet_listener(PacketType.Play.Client.CHAT) # (1)!
def timeout_chat_packet_event(event):
    # Handle the timed-out packet here...
    pass
```

1. Registers `timeout_chat_packet_event` as a timeout listener. An optional `priority` parameter is also available, defaulting to `ListenerPriority.NORMAL`.

See ProtocolLib's documentation for detailed information on timeout listeners.

### Unregistering a Listener

All three decorators attach the same two attributes to the decorated function:

- `.registered_listener` — the `ScriptPacketListener` representing the registered listener.
- `.unregister()` — a convenience method that unregisters the listener.

``` py linenums="1"
chat_packet_event.unregister() # (1)!
```

1. Unregisters the `chat_packet_event` listener. After this call, the function will no longer be called when the packet is intercepted.

???+ tip

    You **do not** need to unregister your packet listeners when your script is stopped/unloaded. PySpigot will handle this for you.

## Protocol Manager Usage

There are several functions available from the protocol manager for registering and unregistering packet listeners:

- `registerPacketListener(function, packet_type)`
- `registerPacketListener(function, packet_type, listener_priority)`: `listener_priority` is a the priority of the listener. This is analogous to EventPriority for Bukkit events.
    - For information on listener priority, see ProtocolLib's [ListenerPriority class](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/events/ListenerPriority.html).
- `unregisterPacketListener(packet_listener)`: Takes a packet listener returned by one of the register functions.
- `createPacket(packet_type)`: Creates and returns a packet with the given type.
    - See the [PacketType class](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/PacketType.html) for a complete list of packet types.
- `sendServerPacket(player, packet)`: Sends a packet to the provided player.
- `broadcastServerPacket(packet)`: Broadcasts a packet to all players on the server.
- `broadcastServerPacket(packet, entity)`: Broadcasts a packet to all players who are monitoring the given entity at the time of the packet being sent.
- `broadcastServerPacket(packet, entity, include_tracker)`: Broadcasts a packet to all players who are monitoring the given entity. `includeTracker` is used to specify if the packet should also be broadcasted to the entity (in addition to the players tracking the entity), e.g. `true` or `false`.
- `broadcastServerPacket(packet, origin, max_observer_distance)`: Broadcasts a packet to all players within a given max observer distance from an origin location (center point).
- `broadcastServerPacket(packet, target_players)`: Broadcasts a packet to a list of target players.

???+ tip

    You **do not** need to unregister your packet listeners when your script is stopped/unloaded. PySpigot will handle this for you.

### Asynchronous Listeners

The protocol manager also supports registering asynchronous and timeout listeners. Asynchronous listeners allow you to delay packet transmission, among other things. Timeout listeners handle packets that time out during asynchronous processing.

The recommended way to register these listeners is with the [`@async_packet_listener`](#async_packet_listener) and [`@timeout_packet_listener`](#timeout_packet_listener) decorators described above.

If you prefer to register them manually, access the asynchronous protocol manager via `asyncManager()`:

``` py linenums="1"
import pyspigot as ps

async_manager = ps.protocol_manager().async()
```

The following functions are available from the asynchronous protocol manager:

- `registerAsyncPacketListener(function, packet_type)`
- `registerAsyncPacketListener(function, packet_type, listener_priority)`
- `registerTimeoutPacketListener(function, packet_type)`
- `registerTimeoutPacketListener(function, packet_type, listener_priority)`
- `unregisterAsyncPacketListener(packet_listener)`: Takes a packet listener returned by one of the register functions.

???+ note

    See ProtocolLib's documentation for more detailed information regarding asynchronous and timeout listeners.

## Code Example

Let's look at the following code that defines and registers a chat packet listener:

``` py linenums="1"
from decorators.protocol import packet_listener # (1)!
from com.comphenix.protocol import PacketType # (2)!

@packet_listener(PacketType.Play.Client.CHAT) # (3)!
def chat_packet_event(event): # (4)!
    packet = event.getPacket() # (5)!
    message = packet.getStrings().read(0) # (6)!
    print(f'Player sent a chat! Their message was: {message}')
```

1. Here, we import the `packet_listener` decorator.

2. Here, we import `PacketType` from ProtocolLib. This is used to define which packet we want to listen for.

3. Here, `@packet_listener(PacketType.Play.Client.CHAT)` registers `chat_packet_event` as a synchronous listener for the `CHAT` packet type.

4. Here, we define the listener function. It has one parameter, `event`, which represents the packet event that was intercepted.

5. Here, we get the packet from the event and assign it to `packet`.

6. Here, we read the chat message string from the packet and print it.

For a complete list of available packet types, see the [PacketType class](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/PacketType.html). Packet types are organized by connection state (`Configuration`, `Handshake`, `Login`, `Play`, `Status`) and further by `Client` or `Server` direction.

### Unregistering a Packet Listener

``` py linenums="1"
chat_packet_event.unregister() # (1)!
```

1. Here, we unregister the packet listener using the `.unregister()` method attached by the decorator.

## Summary

- To define the packet type for your listener, import `PacketType` from ProtocolLib (`com.comphenix.protocol`).
- All packet listeners should be defined as functions in your script that accept a single parameter — the packet event.
- The recommended way to register a listener is with the decorators from `decorators/protocol.py`:
    - `@packet_listener(packet_type)` — synchronous listener.
    - `@async_packet_listener(packet_type)` — asynchronous listener (delays packet transmission).
    - `@timeout_packet_listener(packet_type)` — timeout listener (handles timed-out async packets).
- All three decorators accept an optional `priority` parameter (a `ListenerPriority`, defaulting to `NORMAL`).
- Decorated functions gain a `.registered_listener` attribute (the `ScriptPacketListener`) and an `.unregister()` method.
- Alternatively, listeners can be registered manually via `protocol_manager().registerPacketListener(function, packet_type)` and the async manager via `protocol_manager().asyncManager()`.
- Packet listeners are called **asynchronously** by ProtocolLib. Any code that interacts with the Bukkit API must be run synchronously — use `task_manager().runTask(function)` to bring work back to the main thread.
- You **do not** need to unregister packet listeners when your script stops — PySpigot handles cleanup automatically.