# ProtocolLib Manager

PySpigot includes a manager that interfaces with ProtocolLib if you would like to work with packets in your script.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the protocol manager into your script.

# Protocol Manager Usage

There are several functions available from the protocol manager for registering and unregistering packet listeners:

- `registerPacketListener(function, packet_type)`
- `registerPacketListener(function, packet_type, listener_priority)`: `listener_priority` is a the priority of the listener. This is analogous to EventPriority for Bukkit events.
    - For information on listener priority, see ProtocolLib's [ListenerPriority class](https://ci.dmulloy2.net/job/ProtocolLib/javadoc/com/comphenix/protocol/events/ListenerPriority.html).
- `unregisterPacketListener(function)`

!> All ProtocolLib listeners are called from an *asynchronous* context. In other words, they are called from a thread other than the main server thread. Do not call any Bukkit code from this context, or issues may occur.

## Asynchronous listeners

The protocol manager also supports registering asynchronous listeners. Asynchronous listeners allow you to delay packet transmission, among other things. You may also register timeout listeners that can handle packets that time out on sending.

To get the asynchronous protocol manager, call `async()`. For example,

```python
from dev.magicmq.pyspigot import PySpigot as ps

async_manager = ps.protocol.async()
```

The following functions are avialable for use from the asynchronous manager:

- `registerAsyncListener(function, packet_type)`
- `registerAsyncListener(function, packet_type, listener_priority)`
- `unregisterAsyncListener(function)`
- `registerTimeoutListener(function, packet_type)`
- `registerTimeoutListener(function, packet_type, listener_priority)`
- `unregisterTimeoutListener(function)`


# Code Example

Let's look at the following code that defines and registers a chat packet listener:

```python
from dev.magicmq.pyspigot import PySpigot as ps
from com.comphenix.protocol import PacketType

def chat_packet_event(event):
    packet = event.getPacket()
    message = packet.getStrings().read(0)
    print('Player sent a chat! Their message was: ' + event.getMessage())

ps.protocol.registerPacketListener(chat_packet_event, PacketType.Play.Client.CHAT)
```

On line 1, we import PySpigot as `ps` to utilize the protocol manager (`protocol`).

On line 2, we import `PacketType` from ProtocolLib. This will be used to define which packet we want to listen to.

On line 4, we define `chat_packet_event`, a function that will be called when the packet we are listening to is intercepted. This function has one parameter, `event`, which represents the event that occurred (a chat packet was received).

On lines 4 and 5, we get the packet from the event and read the data from the packet.

All packet listeners must be registered with PySpigot's protocol manager. Registering packet listeners is very similar to 

- The first argument accepts the function that should be called when the event fires.
- The second argument accepts the event that should be listened for.

Therefore, on line 7, we call the listener manager to register our event, passing the function we defined on line 5, `player_chat`, and the event we want to listen for, `AsyncPlayerChatEvent`.

## To summarize: {docsify-ignore}

- To define the packet type for your listener, you must import `PacketType` from ProtocolLib.
- All packet listeners should be defined as functions in your script that accept a single parameter, the packet event (the parameter name can be whatever you like).
- All packet listeners must be registered with PySpigot's packet manager. This can be done in a variety of ways, but the most basic way is by using `registerPacketListener(function, packet_type)`.
- Packet listeners are called *asynchronously*. Any code that interfaces with Bukkit or PySpigot should be run *synchronously*. You can do this by using the task manager (`runTask(function)`)