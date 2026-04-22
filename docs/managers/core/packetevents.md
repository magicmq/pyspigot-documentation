# Working with PacketEvents

???+ warning

    The Packet Events manager is an *optional* manager. This manager should only be accessed if the PacketEvents plugin is present on the server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with [PacketEvents](https://github.com/retrooper/packetevents) if you would like to work with packets in your script. PacketEvents is a cross-platform packet library that works on the Bukkit, BungeeCord, and Velocity platforms, providing a unified API for packet interception and manipulation regardless of the platform you are using.

???+ note

    If you are running PySpigot on Bukkit and are looking for an alternative packet library, PySpigot also supports [ProtocolLib](../bukkit/protocollib.md). If you are running PySpigot on BungeeCord, PySpigot also supports [Protocolize](../bungee/protocolize.md). Unlike these platform-specific managers, the Packet Events manager is available on all three supported platforms.

For instructions on importing the packet events manager into your script, visit the [General Information](../usage.md) page.

## The `packet_listener` Decorator

PySpigot ships with a `decorators/packet_events.py` helper module that provides a Python **decorator** for registering packet listeners. Using the decorator is the recommended way to register packet listeners, as it is cleaner and more Pythonic than calling the packet events manager directly.

???+ warning

    The `decorators/packet_events` module checks whether PacketEvents is available **at import time**. Importing from this module on a server where PacketEvents is not installed will immediately raise a `ScriptRuntimeException`. Only import this module if you know PacketEvents is present.

### Importing

``` py
from decorators.packet_events import packet_listener
```

### Basic Usage

Apply `@packet_listener(PacketType)` to a function to register it as a listener for that packet type. Since PacketEvents is cross-platform, the syntax is the same regardless of platform:

``` py linenums="1"
from decorators.packet_events import packet_listener
from com.github.retrooper.packetevents.protocol.packettype import PacketType # (1)!
from com.github.retrooper.packetevents.wrapper.play.client import WrapperPlayClientChatMessage

@packet_listener(PacketType.Play.Client.CHAT_MESSAGE) # (2)!
def chat_packet(event): # (3)!
    wrapper = WrapperPlayClientChatMessage(event)
    message = wrapper.getMessage()
    print(f'Player sent a chat! Their message was: {message}')
```

1. The packet type class must be imported before it is passed to the decorator.

2. Applying `@packet_listener(PacketType.Play.Client.CHAT_MESSAGE)` registers `chat_packet` as a listener for that packet type. The listener direction (receive vs. send) is determined automatically from the packet type — see [Packet Types](#packet-types) below.

3. The function receives the packet event as its only parameter — a `PacketReceiveEvent` for `*.Client` packet types, or a `PacketSendEvent` for `*.Server` packet types.

### Optional Parameters

- `packet_type` *(required)* — the packet type to listen for.
- `priority` — the listener priority. Defaults to `PacketListenerPriority.NORMAL`. See [Listener Priority](#listener-priority) below.

``` py linenums="1"
from decorators.packet_events import packet_listener
from com.github.retrooper.packetevents.protocol.packettype import PacketType
from com.github.retrooper.packetevents.event import PacketListenerPriority

@packet_listener(PacketType.Play.Client.CHAT_MESSAGE, priority=PacketListenerPriority.HIGH)
def chat_packet(event):
    ...
```

### Unregistering a Listener

When a function is decorated with `@packet_listener`, two attributes are attached to it:

- `.registered_listener` — the `ScriptPacketListener` object representing the registered listener.
- `.unregister()` — a convenience method that unregisters the listener.

``` py linenums="1"
chat_packet.unregister() # (1)!
```

1. Unregisters the `chat_packet` listener. After this call, the function will no longer be called when the packet is intercepted.

???+ tip

    You **do not** need to unregister your packet listeners when your script is stopped/unloaded. PySpigot will handle this for you.

## Packet Events Manager Usage

The following functions are available from the packet events manager:

- `registerPacketListener(function, type)`: Registers a packet listener with the default priority (`PacketListenerPriority.NORMAL`). The listener type (receive or send) is automatically determined from the packet type's side — see the [Packet Types](#packet-types) section below.
- `registerPacketListener(function, type, priority)`: Registers a packet listener with the given priority.
    - For information on listener priorities, see the [Listener Priority](#listener-priority) section below.
- `unregisterPacketListener(packet_listener)`: Unregisters a packet listener. Takes a `ScriptPacketListener` returned by one of the register functions.
- `unregisterPacketListener(function)`: Unregisters any packet listener whose function matches the given function.
- `getPacketEventsAPI()`: Returns the underlying PacketEvents API instance, for direct access to PacketEvents beyond what the manager exposes.

???+ tip

    You **do not** need to unregister your packet listeners when your script is stopped/unloaded. PySpigot will handle this for you.

If you prefer to register packet listeners manually without using the decorator, the functions above are available as an alternative.

## Packet Types

All packet listeners require a packet type, specified using PacketEvents' `PacketType` class. Packet types are organized by their connection state and direction:

- `PacketType.Handshaking.Client` — Packets sent during the initial handshake
- `PacketType.Status.Client` / `PacketType.Status.Server` — Server list ping packets
- `PacketType.Login.Client` / `PacketType.Login.Server` — Login phase packets
- `PacketType.Configuration.Client` / `PacketType.Configuration.Server` — Configuration phase packets (Minecraft 1.20.2+)
- `PacketType.Play.Client` — Packets sent **by the client** (received by the server)
- `PacketType.Play.Server` — Packets sent **by the server** (received by the client)

To use packet types in your script, import `PacketType` from PacketEvents:

``` py
from com.github.retrooper.packetevents.protocol.packettype import PacketType
```

???+ note

    The listener type is determined **automatically** from the packet type you provide:

    - Packet types under `*.Client` (e.g., `PacketType.Play.Client.*`) create a **receive listener**. The function will be called with a `PacketReceiveEvent` — these are packets the server *receives* from the client.
    - Packet types under `*.Server` (e.g., `PacketType.Play.Server.*`) create a **send listener**. The function will be called with a `PacketSendEvent` — these are packets the server *sends* to the client.

    You do not need to specify the direction separately; PySpigot determines this automatically based on the packet type.

For a complete list of available packet types, see PacketEvents' [PacketType source](https://github.com/retrooper/packetevents/blob/2.0/api/src/main/java/com/github/retrooper/packetevents/protocol/packettype/PacketType.java). Packet types are organized by connection state and further by `Client` or `Server` direction.

## Listener Priority

Packet listener priority determines the order in which multiple listeners for the same packet type are invoked. Listeners with lower priority are called before those with higher priority. The available priorities, from lowest to highest, are:

- `PacketListenerPriority.LOWEST` — Called first; should not make final decisions about packet handling
- `PacketListenerPriority.LOW` — Low importance
- `PacketListenerPriority.NORMAL` — Default; recommended for most use cases
- `PacketListenerPriority.HIGH` — High importance
- `PacketListenerPriority.HIGHEST` — Called near last; used for making final determinations about packet handling
- `PacketListenerPriority.MONITOR` — Called last; intended for observing the outcome only — **do not modify packets at this priority**

To use `PacketListenerPriority` in your script, import it from PacketEvents:

``` py
from com.github.retrooper.packetevents.event import PacketListenerPriority
```

## Code Example

Let's look at the following code that defines and registers a chat packet listener. Since PacketEvents provides a unified, platform-agnostic API, the code is identical regardless of the platform you are running on:

=== "Bukkit"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from com.github.retrooper.packetevents.protocol.packettype import PacketType # (2)!
    from com.github.retrooper.packetevents.wrapper.play.client import WrapperPlayClientChatMessage # (3)!

    def chat_packet(event): # (4)!
        wrapper = WrapperPlayClientChatMessage(event) # (5)!
        message = wrapper.getMessage() # (6)!
        print(f'Player sent a chat! Their message was: {message}')

    packet_listener = ps.packet_events.registerPacketListener(chat_packet, PacketType.Play.Client.CHAT_MESSAGE) # (7)!
    ```

    1. Here, we import PySpigot as `ps` to utilize the packet events manager (`packet_events`).

    2. Here, we import `PacketType` from PacketEvents. This is used to specify which packet type we want to listen for.

    3. Here, we import the packet wrapper class for the chat message packet. PacketEvents provides typed wrapper classes for reading and writing packet data.

    4. Here, we define the function `chat_packet`, which will be called when a chat message packet is received from a client. This function takes one parameter, `event`, which is a `PacketReceiveEvent` (since `CHAT_MESSAGE` is a client-side packet).

    5. Here, we wrap the raw event using `WrapperPlayClientChatMessage` to access the packet's data in a typed, structured way.

    6. Here, we call `getMessage()` on the wrapper to retrieve the chat message string that was sent.

    7. Here, we register the packet listener with the packet events manager, passing our function `chat_packet` and the packet type `PacketType.Play.Client.CHAT_MESSAGE`. Because this is a `Client`-side packet, PySpigot automatically creates a receive listener. The returned `ScriptPacketListener` is assigned to `packet_listener` for potential later use.

=== "Velocity"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from com.github.retrooper.packetevents.protocol.packettype import PacketType # (2)!
    from com.github.retrooper.packetevents.wrapper.play.client import WrapperPlayClientChatMessage # (3)!

    def chat_packet(event): # (4)!
        wrapper = WrapperPlayClientChatMessage(event) # (5)!
        message = wrapper.getMessage() # (6)!
        print(f'Player sent a chat! Their message was: {message}')

    packet_listener = ps.packet_events.registerPacketListener(chat_packet, PacketType.Play.Client.CHAT_MESSAGE) # (7)!
    ```

    1. Here, we import PySpigot as `ps` to utilize the packet events manager (`packet_events`).

    2. Here, we import `PacketType` from PacketEvents. This is used to specify which packet type we want to listen for.

    3. Here, we import the packet wrapper class for the chat message packet. PacketEvents provides typed wrapper classes for reading and writing packet data.

    4. Here, we define the function `chat_packet`, which will be called when a chat message packet is received from a client. On Velocity, this intercepts the packet as it travels from the client through the proxy to the backend server. This function takes one parameter, `event`, which is a `PacketReceiveEvent`.

    5. Here, we wrap the raw event using `WrapperPlayClientChatMessage` to access the packet's data in a typed, structured way.

    6. Here, we call `getMessage()` on the wrapper to retrieve the chat message string that was sent.

    7. Here, we register the packet listener with the packet events manager, passing our function `chat_packet` and the packet type `PacketType.Play.Client.CHAT_MESSAGE`. Because this is a `Client`-side packet, PySpigot automatically creates a receive listener. The returned `ScriptPacketListener` is assigned to `packet_listener` for potential later use.

=== "BungeeCord"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from com.github.retrooper.packetevents.protocol.packettype import PacketType # (2)!
    from com.github.retrooper.packetevents.wrapper.play.client import WrapperPlayClientChatMessage # (3)!

    def chat_packet(event): # (4)!
        wrapper = WrapperPlayClientChatMessage(event) # (5)!
        message = wrapper.getMessage() # (6)!
        print(f'Player sent a chat! Their message was: {message}')

    packet_listener = ps.packet_events.registerPacketListener(chat_packet, PacketType.Play.Client.CHAT_MESSAGE) # (7)!
    ```

    1. Here, we import PySpigot as `ps` to utilize the packet events manager (`packet_events`).

    2. Here, we import `PacketType` from PacketEvents. This is used to specify which packet type we want to listen for.

    3. Here, we import the packet wrapper class for the chat message packet. PacketEvents provides typed wrapper classes for reading and writing packet data.

    4. Here, we define the function `chat_packet`, which will be called when a chat message packet is received from a client. On BungeeCord, this intercepts the packet as it travels from the client through the proxy to the backend server. This function takes one parameter, `event`, which is a `PacketReceiveEvent`.

    5. Here, we wrap the raw event using `WrapperPlayClientChatMessage` to access the packet's data in a typed, structured way.

    6. Here, we call `getMessage()` on the wrapper to retrieve the chat message string that was sent.

    7. Here, we register the packet listener with the packet events manager, passing our function `chat_packet` and the packet type `PacketType.Play.Client.CHAT_MESSAGE`. Because this is a `Client`-side packet, PySpigot automatically creates a receive listener. The returned `ScriptPacketListener` is assigned to `packet_listener` for potential later use.

All packet listeners must be registered with PySpigot's packet events manager. Registering a packet listener is similar to registering an event listener, except a packet type is passed instead of an event class:

- The first argument accepts the function that should be called when the packet is intercepted.
- The second argument accepts the packet type to listen for (e.g., `PacketType.Play.Client.CHAT_MESSAGE`).

The `registerPacketListener` function returns a `ScriptPacketListener` representing the registered packet listener. This object can be used to unregister the listener later if needed.

???+ info

    PacketEvents provides wrapper classes for reading and writing structured packet data (e.g., `WrapperPlayClientChatMessage`). These wrappers are located under `com.github.retrooper.packetevents.wrapper.*`. For a full listing of available wrapper classes and their methods, refer to the [PacketEvents documentation](https://github.com/retrooper/packetevents).

### Unregistering a Packet Listener

Continuing the above code example:

``` py linenums="1"
ps.packet_events.unregisterPacketListener(packet_listener) # (1)!
```

1. Here, we unregister the packet listener by passing the `ScriptPacketListener` object that was returned when the listener was registered.

## Summary

- To use packet types, import `PacketType` from `com.github.retrooper.packetevents.protocol.packettype`.
- All packet listeners should be defined as functions in your script that accept a single parameter — the packet event (either a `PacketReceiveEvent` or `PacketSendEvent`).
- The listener type (receive vs. send) is determined **automatically** from the packet type — no need to specify direction manually.
    - Packet types under `*.Client` (e.g., `PacketType.Play.Client.*`) produce a `PacketReceiveEvent`.
    - Packet types under `*.Server` (e.g., `PacketType.Play.Server.*`) produce a `PacketSendEvent`.
- The recommended way to register a packet listener is via the `@packet_listener(PacketType)` decorator from `decorators/packet_events.py`.
- Decorated functions gain a `.registered_listener` attribute (the `ScriptPacketListener`) and an `.unregister()` method for easy cleanup.
- Alternatively, packet listeners can be registered manually via the packet events manager using `registerPacketListener(function, type)`.
- To read or write packet data, use PacketEvents' typed wrapper classes (e.g., `WrapperPlayClientChatMessage`), which are located under `com.github.retrooper.packetevents.wrapper.*`.
- You **do not** need to unregister packet listeners when your script stops — PySpigot handles cleanup automatically.
