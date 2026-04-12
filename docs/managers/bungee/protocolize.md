# Working with Protocolize

???+ warning

    The Protocol manager is an *optional* manager. This manager should only be accessed if the Protocolize plugin is present on the BungeeCord proxy server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with Protocolize if you would like to work with packets in your script.

For instructions on importing the protocol manager into your script, visit the [General Information](../usage.md) page.

## Protocol Manager Usage

There are several functions available from the protocol manager for registering and unregistering packet listeners:

- `registerPacketListener(receive_function, send_function, packet, direction)`: Registers a packet listener with default priority.
- `registerPacketListener(receive_function, send_function, packet, direction, priority)`: Registers a packet listener with the given priority. `priority` is an integer representing the listener's priority relative to other listeners registered for the same packet.
- `unregisterPacketListener(packet_listener)`: Unregisters a packet listener. Takes a `ScriptPacketListener` returned by one of the register functions.
- `unregisterPacketListener(function)`: Unregisters any packet listeners whose receive or send function matches the given function. Note that multiple listeners may be unregistered if the same function is associated with more than one listener.
- `unregisterPacketListener(function, packet)`: Unregisters the packet listener whose receive or send function matches the given function and whose packet type matches the given packet class.
- `sendPacket(player_uuid, packet)`: Sends a Protocolize packet (an `AbstractPacket`) to the player with the given UUID.
- `sendPacket(player_uuid, packet)`: Sends a generic BungeeCord packet (a `DefinedPacket`) to the player with the given UUID.

???+ tip

    You **do not** need to unregister your packet listeners when your script is stopped/unloaded. PySpigot will handle this for you.

## Packet Direction

All packet listeners require a `Direction`, which specifies which side of the proxy the listener should operate on:

- `Direction.UPSTREAM`: The connection between the proxy and the **client**. All packets coming in from and going out to the client travel on this connection.
- `Direction.DOWNSTREAM`: The connection between the proxy and the **backend Minecraft server**. All packets coming in from and going out to the backend server travel on this connection.

To use the `Direction` enum in your script, import it from Protocolize:

``` py
from dev.simplix.protocolize.api import Direction
```

## Code Example

Let's look at the following code that defines and registers a chat packet listener:

``` py linenums="1"
import pyspigot as ps # (1)!
from dev.simplix.protocolize.api import Direction # (2)!
from net.md_5.bungee.protocol.packet import Chat # (3)!

def chat_received(event): # (4)!
    packet = event.packet() # (5)!
    print(f'A chat packet was received from the client!')

def chat_sent(event): # (6)!
    packet = event.packet() # (7)!
    print(f'A chat packet was sent to the client!')

packet_listener = ps.protocol.registerPacketListener(chat_received, chat_sent, Chat, Direction.UPSTREAM) # (8)!
```

1. Here, we import PySpigot as `ps` to utilize the protocol manager (`protocol`).

2. Here, we import `Direction` from Protocolize. This will be used to specify which side of the proxy the listener should listen on.

3. Here, we import the `Chat` packet class from BungeeCord's protocol package. This will be used to define which packet type we want to listen for.

4. Here, we define the function `chat_received`, a function that will be called when the chat packet is received (i.e., arrives from the client). This function has one parameter, `event`, which represents the packet receive event.

5. Here, we call `event.packet()` to retrieve the received packet.

6. Here, we define the function `chat_sent`, a function that will be called when the chat packet is sent (i.e., is transmitted to the client). This function also has one parameter, `event`, which represents the packet send event.

7. Here, we call `event.packet()` to retrieve the packet being sent.

8. Here, we call the protocol manager to register the packet listener, passing the receive function `chat_received`, the send function `chat_sent`, the packet class `Chat`, and the direction `Direction.UPSTREAM`. The returned value is assigned to `packet_listener`.

All packet listeners must be registered with PySpigot's protocol manager. Unlike the ProtocolLib manager, which uses a single function for intercepted packets, Protocolize requires **two separate functions**: one called when the packet is received, and one called when it is sent.

- The first argument accepts the function that should be called when the packet is received.
- The second argument accepts the function that should be called when the packet is sent.
- The third argument accepts the packet class to listen for.
- The fourth argument accepts the direction (`Direction.UPSTREAM` or `Direction.DOWNSTREAM`).

Both functions are required — neither can be omitted.

The `registerPacketListener` function returns a `ScriptPacketListener`, which represents the packet listener that was registered. This can be used to unregister the packet listener at a later time.

### Unregistering a Packet Listener

Continuing the above code example:

``` py linenums="1"
ps.protocol.unregisterPacketListener(packet_listener) # (1)!
```

1. Here, we unregister the packet listener by passing the `ScriptPacketListener` object we assigned earlier when registering the packet listener.

## Summary

- To define the packet direction, import `Direction` from `dev.simplix.protocolize.api`.
- All packet listeners require **two** functions: one for when the packet is received (`PacketReceiveEvent`), and one for when it is sent (`PacketSendEvent`). Neither can be `None`.
- All packet listeners must be registered with PySpigot's protocol manager using `registerPacketListener(receive_function, send_function, packet, direction)`.
- `Direction.UPSTREAM` listens on the proxy-client connection; `Direction.DOWNSTREAM` listens on the proxy-backend server connection.
- When registering a packet listener, the register functions return a `ScriptPacketListener`, which can be used to unregister the listener.
