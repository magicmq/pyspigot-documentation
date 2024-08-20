# Protocol Manager

!> The Protocol manager is an *optional* manager. This manager should only be accessed if the ProtocolLib plugin is present on the server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with ProtocolLib if you would like to work with packets in your script.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the protocol manager into your script.

!> All packet listeners are called from an *asynchronous* context. In other words, they are called from a thread other than the main server thread. Do not call any Bukkit or PySpigot API from this context, or issues may occur.

# Protocol Manager Usage

There are several functions available from the protocol manager for registering and unregistering packet listeners:

- `registerPacketListener(function, packet_type)`
- `registerPacketListener(function, packet_type, listener_priority)`: `listener_priority` is a the priority of the listener. This is analogous to EventPriority for Bukkit events.
    - For information on listener priority, see ProtocolLib's [ListenerPriority class](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/events/ListenerPriority.html).
- `unregisterPacketListener(packet_listener)`: Takes a packet listener returned by one of the register functions.
- `createPacket(packet_type)`: Creates and returns a packet with the given type. See the [PacketType class](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/PacketType.html).
- `sendServerPacket(player, packet)`: Sends a packet to the provided player.
- `broadcastServerPacket(packet)`: Broadcasts a packet to all players on the server.
- `broadcastServerPacket(packet, entity)`: Broadcasts a packet to all players who are monitoring the given entity at the time of the packet being sent.
- `broadcastServerPacket(packet, entity, include_tracker)`: Broadcasts a packet to all players who are monitoring the given entity. `includeTracker` is used to specify if the packet should also be broadcasted to the entity (in addition to the players tracking the entity), e.g. `true` or `false`.
- `broadcastServerPacket(packet, origin, max_observer_distance)`: Broadcasts a packet to all players within a given max observer distance from an origin location (center point).
- `broadcastServerPacket(packet, target_players)`: Broadcasts a packet to a list of target players.

?> You **do not** need to unregister your packet listeners when your script is stopped/unloaded. PySpigot will handle this for you.

## Asynchronous Listeners

The protocol manager also supports registering asynchronous listeners. Asynchronous listeners allow you to delay packet transmission, among other things. You may also register timeout listeners that can handle packets that time out on sending.

To get the asynchronous protocol manager, call `async()`. For example,

```python
import pyspigot as ps

async_manager = ps.protocol.async()
```

The following functions are avialable for use from the asynchronous protocol manager:

- `registerAsyncPacketListener(function, packet_type)`
- `registerAsyncPacketListener(function, packet_type, listener_priority)`
- `registerTimeoutPacketListener(function, packet_type)`
- `registerTimeoutPacketListener(function, packet_type, listener_priority)`
- `unregisterAsyncPacketListener(packet_listener)`: Takes a packet listener returned by one of the register functions.

?> See ProtocolLib's documentation for more detailed infromation regarding asynchronous and timeout listeners.

# Code Example

Let's look at the following code that defines and registers a chat packet listener:

```python
import pyspigot as ps
from com.comphenix.protocol import PacketType

def chat_packet_event(event):
    packet = event.getPacket()
    message = packet.getStrings().read(0)
    print('Player sent a chat! Their message was: ' + event.getMessage())

packet_listener = ps.protocol.registerPacketListener(chat_packet_event, PacketType.Play.Client.CHAT)
```

On line 1, we import PySpigot as `ps` to utilize the protocol manager (`protocol`).

On line 2, we import `PacketType` from ProtocolLib. This will be used to define which packet we want to listen to.

On line 4, we define `chat_packet_event`, a function that will be called when the packet we are listening to is intercepted. This function has one parameter, `event`, which represents the event that occurred (a chat packet was received).

On lines 4 and 5, we get the packet from the event and read the data from the packet.

All packet listeners must be registered with PySpigot's protocol manager. Registering packet listeners is very similar to 

- The first argument accepts the function that should be called when the event fires.
- The second argument accepts the event that should be listened for.

The `registerPacketListener` function returns a `ScriptPacketListener`, which represents the packet listener that was registered. This can be used to unregister the packet listener at a later time.

Therefore, on line 7, we call the protocol manager to register the packet listener, passing the function we defined on line 5, `chat_packet_event`, and the packet we want to listen for, `PacketType.Play.Client.CHAT`, and assign the returned value to `packet_listener`.

## To summarize: {docsify-ignore}

- To define the packet type for your listener, you must import `PacketType` from ProtocolLib.
- All packet listeners should be defined as functions in your script that accept a single parameter, the packet event (the parameter name can be whatever you like).
- All packet listeners must be registered with PySpigot's packet manager. This can be done in a variety of ways, but the most basic way is by using `registerPacketListener(function, packet_type)`.
- Packet listeners are called *asynchronously*. Any code that interfaces with Bukkit or PySpigot should be run *synchronously*. You can do this by using the task manager (`runTask(function)`)
- When registering a packet listener, the register functions all return a `ScriptPacketListener`, which can be used to unregister the listener.