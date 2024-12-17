# Working with ProtocolLib

???+ warning

    The Protocol manager is an *optional* manager. This manager should only be accessed if the ProtocolLib plugin is present on the server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with ProtocolLib if you would like to work with packets in your script.

For instructions on importing the placeholder manager into your script, visit the [General Information](usage.md) page.

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

The protocol manager also supports registering asynchronous listeners. Asynchronous listeners allow you to delay packet transmission, among other things. You may also register timeout listeners that can handle packets that time out on sending.

To get the asynchronous protocol manager, call `async()`. For example,

``` py linenums="1"
import pyspigot as ps

async_manager = ps.protocol.async()
```

The following functions are avialable for use from the asynchronous protocol manager:

- `registerAsyncPacketListener(function, packet_type)`
- `registerAsyncPacketListener(function, packet_type, listener_priority)`
- `registerTimeoutPacketListener(function, packet_type)`
- `registerTimeoutPacketListener(function, packet_type, listener_priority)`
- `unregisterAsyncPacketListener(packet_listener)`: Takes a packet listener returned by one of the register functions.

??? note

    See ProtocolLib's documentation for more detailed infromation regarding asynchronous and timeout listeners.

## Code Example

Let's look at the following code that defines and registers a chat packet listener:

``` py linenums="1"
import pyspigot as ps # (1)!
from com.comphenix.protocol import PacketType # (2)!

def chat_packet_event(event): # (3)!
    packet = event.getPacket() # (4)!
    message = packet.getStrings().read(0) # (5)!
    print(f'Player sent a chat! Their message was: {message}')

packet_listener = ps.protocol.registerPacketListener(chat_packet_event, PacketType.Play.Client.CHAT) # (6)!
```

1. Here, we import PySpigot as `ps` to utilize the protocol manager (`protocol`).

2. Here, we import `PacketType` from ProtocolLib. This will be used to define which packet we want to listen to.

3. Here, we define the function `chat_packet_event`, a function that will be called when the packet we are listening to is intercepted. This function has one parameter, `event`, which represents the event that occurred (the chat packet that was received).

4. Here, we get the packet that was intercepted and assign it to `packet`.

5. Here, we read the data (the chat message) from the packet, and subsequently print the message that was intercepted on the next line.

6. Here, we call the protocol manager to register the packet listener, passing the function we defined on line 5, `chat_packet_event`, and the packet we want to listen for, `PacketType.Play.Client.CHAT`, and assign the returned value to `packet_listener`.

All packet listeners must be registered with PySpigot's protocol manager. Registering a packet listener is very similar to registering an event listener with PySpigot's listener manager, except that a packet type is passed to the register function insead of an event.

- The first argument accepts the function that should be called when the packet is intercepted.
- The second argument accepts the packet type to listen to.

The `registerPacketListener` function returns a `ScriptPacketListener`, which represents the packet listener that was registered. This can be used to unregister the packet listener at a later time.

For a complete list of available packet types, see the [PacketType class](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/PacketType.html). Packet types are broken up into several types: `Configuration`, `Handshake`, `Login`, `Play`, and `Status`, based on what stage of gameplay they are sent/received. Each of these categories is further subdivided into `Client` and `Server` based on whether the packet originated from the client or the server.

### Unregistering a Packet Listener

Continuing the above code example:

``` py linenums="1"
ps.protocol.unregisterPacketListener(listener) # (1)!
```

1. Here, we unregister the packet listener by passing the `ScriptPacketListener` object we assigned earlier when registering the packet listener.

## Summary

- To define the packet type for your listener, you must import `PacketType` from ProtocolLib.
- All packet listeners should be defined as functions in your script that accept a single parameter, the packet event (the parameter name can be whatever you like).
- All packet listeners must be registered with PySpigot's packet manager. This can be done in a variety of ways, but the most basic way is by using `registerPacketListener(function, packet_type)`.
- Packet listeners are called *asynchronously*. Any code that interfaces with Bukkit or PySpigot should be run *synchronously*. You can do this by using the task manager (`runTask(function)`)
- When registering a packet listener, the register functions all return a `ScriptPacketListener`, which can be used to unregister the listener.