# Working with Protocolize

???+ warning

    The Protocol manager is an *optional* manager. This manager should only be accessed if the Protocolize plugin is present on the BungeeCord proxy server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with Protocolize if you would like to work with packets in your script.

For instructions on importing the protocol manager into your script, visit the [General Information](../usage.md) page.

## Protocol Decorators

PySpigot ships with a `decorators/protocol.py` helper module that provides Python **decorators** for registering packet listeners. Using the decorators is the recommended way to register packet listeners, as it is cleaner and more Pythonic than calling the protocol manager directly.

???+ warning

    The `decorators/protocol` module checks whether Protocolize is available **at import time**. Importing from this module on a server where Protocolize is not installed will immediately raise a `ScriptRuntimeException`. Only import this module if you know Protocolize is present.

### Importing

``` py
from decorators.protocol import packet_listener_receive
from decorators.protocol import packet_listener_send  # (1)!
```

1. Only needed if you prefer to decorate the send function first. See [`@packet_listener_send`](#packet_listener_send) below.

### Registration Pattern

Protocolize packet listeners require **two** functions: one called when the packet is received, and one called when it is sent. The decorators reflect this two-sided model — you decorate one function first, which exposes a complementary decorator to attach the second. The listener is not registered until **both** sides are attached.

Both receive and send functions accept a single `event` parameter:

- For receive functions, `event` is a `PacketReceiveEvent`. Call `event.packet()` to retrieve the received packet.
- For send functions, `event` is a `PacketSendEvent`. Call `event.packet()` to retrieve the packet being sent.

### `@packet_listener_receive`

Use `@packet_listener_receive` to start from the receive function. Once applied, the decorated function gains a `.send` attribute that acts as a decorator to register the send function. The listener is fully registered when `.send` is applied.

``` py linenums="1"
from decorators.protocol import packet_listener_receive
from dev.simplix.protocolize.api import Direction
from net.md_5.bungee.protocol.packet import Chat

@packet_listener_receive(Chat, Direction.UPSTREAM) # (1)!
def on_receive(event): # (2)!
    packet = event.packet()
    print('Chat packet received from client!')

@on_receive.send # (3)!
def on_send(event): # (4)!
    packet = event.packet()
    print('Chat packet sent to client!')
```

1. `@packet_listener_receive(Chat, Direction.UPSTREAM)` registers `on_receive` as the receive-side handler for `Chat` packets traveling on the proxy-client connection. An optional `priority` parameter (integer, default `0`) controls listener ordering relative to other listeners for the same packet.

2. The receive function takes a single `event` parameter — a `PacketReceiveEvent`. Call `event.packet()` to retrieve the received packet.

3. `@on_receive.send` registers `on_send` as the send-side handler for the same listener. The listener is fully registered at this point.

4. The send function takes a single `event` parameter — a `PacketSendEvent`. Call `event.packet()` to retrieve the packet being sent.

### `@packet_listener_send`

Use `@packet_listener_send` to start from the send function instead. Once applied, the decorated function gains a `.receive` attribute to register the receive function.

``` py linenums="1"
from decorators.protocol import packet_listener_send
from dev.simplix.protocolize.api import Direction
from net.md_5.bungee.protocol.packet import Chat

@packet_listener_send(Chat, Direction.UPSTREAM) # (1)!
def on_send(event):
    packet = event.packet()
    print('Chat packet sent to client!')

@on_send.receive # (2)!
def on_receive(event):
    packet = event.packet()
    print('Chat packet received from client!')
```

1. `@packet_listener_send(Chat, Direction.UPSTREAM)` registers `on_send` as the send-side handler.

2. `@on_send.receive` registers `on_receive` as the receive-side handler and completes the registration.

### Optional Parameters

Both decorators accept an optional `priority` parameter — an integer defaulting to `0`. Higher values indicate higher priority relative to other listeners for the same packet type:

``` py
@packet_listener_receive(Chat, Direction.UPSTREAM, priority=10)
def on_receive(event):
    ...
```

### Unregistering a Listener

After both sides of a listener are registered, both functions gain two attributes:

- `.listener` — the `ScriptPacketListener` representing the registered listener.
- `.unregister()` — a convenience method that unregisters the listener.

``` py linenums="1"
on_receive.unregister() # (1)!
```

1. Unregisters the listener. After this call, neither `on_receive` nor `on_send` will be called for intercepted packets. The same effect can be achieved via `on_send.unregister()`.

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

## Protocol Manager Usage

If you prefer to register packet listeners manually without using the decorators, the protocol manager functions are available as an alternative:

- `registerPacketListener(receive_function, send_function, packet, direction)`: Registers a packet listener with default priority (`0`). Returns a `ScriptPacketListener`.
- `registerPacketListener(receive_function, send_function, packet, direction, priority)`: Registers a packet listener with the given priority. `priority` is an integer controlling the listener's order relative to other listeners for the same packet. Returns a `ScriptPacketListener`.
- `unregisterPacketListener(packet_listener)`: Unregisters a packet listener. Takes a `ScriptPacketListener` returned by one of the register functions.
- `unregisterPacketListener(function)`: Unregisters any packet listeners whose receive or send function matches the given function. Note that multiple listeners may be unregistered if the same function is associated with more than one listener.
- `unregisterPacketListener(function, packet)`: Unregisters the packet listener whose receive or send function matches the given function and whose packet type matches the given packet class.
- `sendPacket(player_uuid, packet)`: Sends a Protocolize packet (an `AbstractPacket`) to the player with the given UUID.
- `sendPacket(player_uuid, packet)`: Sends a generic BungeeCord packet (a `DefinedPacket`) to the player with the given UUID.

???+ tip

    You **do not** need to unregister your packet listeners when your script is stopped/unloaded. PySpigot will handle this for you.

## Code Example

Let's look at the following code that defines and registers a chat packet listener:

``` py linenums="1"
from decorators.protocol import packet_listener_receive # (1)!
from dev.simplix.protocolize.api import Direction # (2)!
from net.md_5.bungee.protocol.packet import Chat # (3)!

@packet_listener_receive(Chat, Direction.UPSTREAM) # (4)!
def chat_received(event): # (5)!
    packet = event.packet() # (6)!
    print(f'A chat packet was received from the client!')

@chat_received.send # (7)!
def chat_sent(event): # (8)!
    packet = event.packet() # (9)!
    print(f'A chat packet was sent to the client!')
```

1. Here, we import the `packet_listener_receive` decorator.

2. Here, we import `Direction` from Protocolize. This is used to specify which side of the proxy the listener should operate on.

3. Here, we import the `Chat` packet class from BungeeCord's protocol package. This defines which packet type to listen for.

4. Here, `@packet_listener_receive(Chat, Direction.UPSTREAM)` registers `chat_received` as the receive-side handler for `Chat` packets on the proxy-client connection.

5. Here, we define `chat_received`. It takes a single `event` parameter, which is a `PacketReceiveEvent`.

6. Here, we call `event.packet()` to retrieve the received packet.

7. Here, `@chat_received.send` registers `chat_sent` as the send-side handler for the same listener, completing registration.

8. Here, we define `chat_sent`. It takes a single `event` parameter, which is a `PacketSendEvent`.

9. Here, we call `event.packet()` to retrieve the packet being sent.

All packet listeners must be registered with a receive function and a send function — neither can be omitted. Unlike the ProtocolLib manager (which uses a single function for intercepted packets), Protocolize requires two separate functions.

### Unregistering a Packet Listener

Continuing the above code example:

``` py linenums="1"
chat_received.unregister() # (1)!
```

1. Here, we unregister the packet listener using the `.unregister()` method attached by the decorator. The same result can be achieved by calling `chat_sent.unregister()`.

## Summary

- To define the packet direction, import `Direction` from `dev.simplix.protocolize.api`.
- All packet listeners require **two** functions: one for when the packet is received (`PacketReceiveEvent`), and one for when it is sent (`PacketSendEvent`). Neither can be omitted.
- Both functions accept a single `event` parameter. Call `event.packet()` to retrieve the intercepted packet.
- The recommended way to register a listener is with the decorators from `decorators/protocol.py`:
    - `@packet_listener_receive(packet_class, direction)` — decorate the receive function first, then apply `@<function>.send` to the send function.
    - `@packet_listener_send(packet_class, direction)` — decorate the send function first, then apply `@<function>.receive` to the receive function.
- Both decorators accept an optional `priority` parameter (integer, default `0`). The listener is not registered until both sides are attached.
- After registration, both functions gain a `.listener` attribute (the `ScriptPacketListener`) and an `.unregister()` method.
- `Direction.UPSTREAM` listens on the proxy-client connection; `Direction.DOWNSTREAM` listens on the proxy-backend server connection.
- Alternatively, listeners can be registered manually via `protocol_manager().registerPacketListener(receive_function, send_function, packet, direction)`.
- You **do not** need to unregister packet listeners when your script stops — PySpigot handles cleanup automatically.
