# Plugin Messaging

???+ note

    This manager is available on the Bukkit/Paper version of PySpigot only.

Plugin messaging is a powerful way for plugins to communicate with clients. Additionally, when the Minecraft server is running behind a proxy such as BungeeCord or Velocity, plugin messaging can also be used to communicate directly with the proxy server.

PySpigot includes a plugin messaging manager that makes it easier for scripts and projects to send plugin messages and register plugin message listeners.

For instructions on importing the messaging manager into your script, visit the [General Information](usage.md) page.

## General Description

Originally, plugin messaging was implemented so that server-sided plugins could send any kind of data to client-sided mods. Before, such messages would have needed to implement custom packets, but the problem with this is that the Minecraft client would get disconnected from servers using those custom packets with an incomprehensible error message, because the client would be unable to read that packet. To fix this issue, Mojang introduced the "Custom Plugin Message Packets".

The anatomy of a Plugin Message Packet is as such:

1. The name of the "channel" (also named "tag" in BungeeCord) the plugin message transits through
2. The size of the plugin message
3. The actual data/payload of the message

Note that the messaging manager handles all of this under the hood, and all that is needed from the user end to send a mesasge is the player to send the plugin message to, the channel, and the message payload.

### The BungeeCord Channel

BungeeCord exposes a special plugin channel to the backend/Bukkit servers called "BungeeCord". Within the BungeeCord channel, there are several message types that can be leveraged to perform actions or fetch data about a particular server, or even send custom data. For a complete list of message types and what they do, see [this table](https://docs.papermc.io/paper/dev/plugin-messaging/#plugin-message-types).

The messaging manager includes a special function called `sendBungeeMessage` for sending messages on the BungeeCord channel. See the section below on [sending plugin messages](#sending-plugin-messages).

## Messaging Manager Usage

The messaging manager can be used to both send and listen to (receive) plugin messages:

### Sending Plugin Messages

The messaging manager provides three functions that can be used to send plugin messages:

- `sendBungeeMessage(player, message_type, *payload)`: Used to send traditional messages on the "BungeeCord" channel, such as a message to send a player to another server, or a message to retrieve the player count of another server.
    - `player`: The player to use when sending the message. Note that at least one player must be online in order to send a plugin message.
    - `message_type`: The type of message to send. For a complete list, see [this table](https://docs.papermc.io/paper/dev/plugin-messaging/#plugin-message-types).
    - `*payload`: Any number of arguments which are send as the message payload. Typically, this is a server name or player name (or both) and depends on the message type.
- `sendMessage(player, channel, *payload)`: Used to send a plugin message to any channel.
    - `player`: The player to use when sending the message.
    - `channel`: The channel to send the message on.
    - `*payload`: Any number of arguments which are send as the message payload. Allowed payload types include the Java types `String`, `Integer`, `Short`, `Byte`, `Boolean`, and `byte[]` (a byte array).
- `sendRawMessage(player, channel, payload)`: This is equivalent to `sendMessage`, except that the payload should be passed as a raw array of bytes.
    - `player`: The player to use when sending the message.
    - `channel`: The channel to send the message on.
    - `*payload`: Any number of arguments which are send as the message payload. Allowed payload types include the Java types `String`, `Integer`, `Short`, `Byte`, `Boolean`, and `byte[]` (a byte array).

#### Sending a player to another server on the proxy

The following example demonstrates how to use the message manager to send a player to another server on the proxy, using the "BungeeCord" channel:

``` py linenums="1"
import pyspigot as ps # (1)!

from org.bukkit import Bukkit

messaging = ps.message_manager()

player = Bukkit.getPlayer('Player') # (2)!

messaging.sendBungeeMessage(player, 'Connect', 'lobby') # (3)!
```

1. Here, we import PySpigot as `ps` to access the messaging manager (`message_manager()`).

2. Here, we arbitrarily get a player by the name "Player". Note that this is purely arbitrary, you could get a player via any means (such as if a player triggers an event).

3. Here, we send a plugin message using the player. We specify a message type of "Connect" (the message type which connects the player to another server) and then specify the server name as the payload ("lobby" in this case).

#### The Forward Message Type

The "BungeeCord" channel includes a special message type called "Forward" that allows you to send a custom message to other servers on the proxy. The "Forward" message type is thus very useful when something happens on one server and the information should be shared with all other servers. For example, if a player is banned on one server, you may want to transmit this information to all servers on the proxy so that the player can be banned on those servers as well.

???+ warning

    All message types, including the "Forward" message type, require a player to be online on **both** the sending **and** receiving servers. Consider using alternative messaging systems (such as redis) if dispatch and receipt of the information is critical.

The following example demonstrates how to use the "Forward" message type to communicate to another server that a player was banned:

``` py linenums="1"
import pyspigot as ps # (1)!

from org.bukkit import Bukkit

messaging = ps.message_manager()

player = Bukkit.getPlayer('Player') # (2)!
banned_player = ... # (3)!

messaging.sendBungeeMessage(player, 'Forward', 'ALL', 'InternalChannel', banned_player.getUniqueId().toString()) # (4)!
```

1. Here, we import PySpigot as `ps` to access the messaging manager (`message_manager()`).

2. Here, we arbitrarily get a player by the name "Player". Note that in this example, it need not be the same player that was banned. This player is simply being used to send the plugin message. It could be any online player.

3. Here, we fetch the player that was banned. 

4. There's a lot happening on this line: First, we use `player` to send the plugin message. Next, we specify "Forward" as the message type. Next, this message type requires we specify the server to send the message to, as well as an "internal channel". `ALL` denotes that we want the message to be sent to ALL servers (but you could also specify a specific server name here). We use `InternalChannel` for the channel name, but any name would do. Finally, we send the actual payload, which is the banned player's UUID, converted to a String with `toString()`.

???+ note

    The "Forward" message type allows for sending any type and amount of payload to the internal channel. For example, any of the following would work:

    ``` py
    messaging.sendBungeeMessage(player, 'Forward', 'ALL', 'InternalChannel', 'Message 1', 'Message 2') # Payload is "Message 1" and "Message 2"
    messaging.sendBungeeMessage(player, 'Forward', 'ALL', 'InternalChannel', 'PySpigot is an awesome plugin', 42) # Payload is "PySpigot is an awesome plugin" and "42"
    messaging.sendBungeeMessage(player, 'Forward', 'ALL', 'InternalChannel', 2, 4, 8, 16, 32) # Payload is "2", "4", "8", "16", and "32"
    ```

#### Sending a message to a custom channel

The following example demonstrates how to send a message to a custom channel:

``` py linenums="1"
import pyspigot as ps # (1)!

from org.bukkit import Bukkit

messaging = ps.message_manager()

player = Bukkit.getPlayer('Player') # (2)!

messaging.sendMessage(player, 'custom_channel', 'Payload') # (3)!
```

1. Here, we import PySpigot as `ps` to access the messaging manager (`message_manager()`).

2. Here, we fetch an online player to send the message. Again, this is entirely arbitrary, any online player could be used.

3. Finally, we send the message containing the payload "Payload" to the channel with the name "custom_channel".

### Receiving Messages

When sending messages that respond with something meaningful, we also need a plugin message listener to listen for incoming plugin messages. The messaging manager provides several functions for registering and unregistering plugin message listeners:

- `registerListener(function, channel)`: Registers a plugin message listener to listen on the given channel. When a message is received, the function is called.
    - `function`: The function that should be called when a message is received on the channel. This function should accept three arguments: `channel`, the channel the message was received on, `player`, the source of the message, and `message`: the message payload itself.
    - `channel`: The channel to listen on
    - Returns a `ScriptPluginMessageListener` object, which represents a registered plugin message listener. Can be used later to unregister the listener if desired.
- `unregisterListener(listener)`: Unregisters a previously registered plugin message listener.
    - `listener`: The listener to unregister
- `unregisterListeners(script)`: Unregister all plugin message listeners belonging to a script.
    - `script`: The script whose plugin message listeners should be unregistered

???+ note

    When a script/project is stopped, all its plugin message listeners are automatically unregistered. You do not need to unregister them yourself.

#### Receiving a player count message

Suppose a plugin message was sent with the type "PlayerCount", a message type which can be used to fetch the player count of another server on the proxy: `sendBungeeMessage(player 'PlayerCount', 'server1')`. We expect to receive a reply stating the number of players on the server "server1".

The following example demonstrates how to handle and parse the response:

```py linenums="1"
import pyspigot as ps # (1)!
from com.google.common.io import ByteStreams # (2)!
from org.bukkit import Bukkit

messaging = ps.message_manager()

def message_received(channel, player, message): # (3)!
    input = ByteStreams.newDataInput(message) # (4)!

    subchannel = input.readUTF() # (5)!
    if subchannel == 'PlayerCount': # (6)!
        server = in.readUTF() # (7)!
        player_count = in.readInt() # (8)!
        print('The player count of server ' + server + ' is ' + str(player_count))

listener = messaging.registerListener(message_received, 'BungeeCord') # (9)!

player = Bukkit.getPlayer('Player') # (10)!
messaging.sendBungeeMessage(player, 'PlayerCount', 'lobby') # (11)!
```

1. Here, we import PySpigot as `ps` to access the messaging manager (`message_manager()`).

2. Here, we import the Java class `ByteStreams`, for use when reading the received message.

3. Here we declare a new function that accepts a channel, player, and message. This function serves as the listener function and is called when a message is received. `channel` represents the channel the message was received on. `player` is the player the message was sent through. `message` is a byte array representing the actual payload of the message.

4. Since the message payload is a raw byte array, we need to use the `ByteStreams` class to decode it. Here, we create a new data input to read the message.

5. Here, we read the first part of the message, which is the subchannel of the message, the same thing as the message type.

6. We are interested in receiving a message regarding the player count, so we check if the subchannel is "PlayerCount".

7. Next, we read the server whose player count we are fetching. This is a string, so we call `in.readUTF()`.

8. Finally, we read the actual player count of the server. This is an int, so we call `in.readInt()`.

9. Here, we register a new plugin message listener, passing the listener function we defined earlier, as well as the channel we want to listen on, "BungeeCord" in this case.

9. Here, we fetch an online player to send the message. Again, this is entirely arbitrary, any online player could be used.

10. Here, we send a plugin message using the player. We specify a message type of "PlayerCount" (the message type which fetches the player count of a particular server) and then specify the server name as the payload ("lobby" in this case).

#### Receiving a "Forward" message

Suppose we sent a plugin message of the "Forward" type, and we want to receive this message on another server. We sent the message with the following: `sendBungeeMessage(player, 'Forward', 'ALL', 'InternalChannel', banned_player.getUniqueId().toString())`.

The following example demonstrates how to handle and parse the response:

```py linenums="1"
import pyspigot as ps # (1)!
from com.google.common.io import ByteStreams # (2)!
from java.io import DataInputStream # (3)!
from java.io import ByteArrayInputStream # (4)!
from org.bukkit import Bukkit

messaging = ps.message_manager()

def message_received(channel, player, message): # (5)!
    input = ByteStreams.newDataInput(message) # (6)!

    subchannel = input.readUTF() # (7)!
    if subchannel == 'InternalChannel': # (8)!
        length = in.readShort() # (9)!
        msg_bytes = bytearray(length) # (10)!
        in.readFully(msg_bytes) # (11)!

        msg_in = DataInputStream(ByteArrayInputStream(msg_bytes)) # (12)!
        message = msg_in.readUTF()  # (13)!
        print('Message received: ' + message)
```

1. Here, we import PySpigot as `ps` to access the messaging manager (`message_manager()`).

2. Here, we import the Java class `ByteStreams`, for use when reading the received message.

3. Here, we import the Java class `DataInputStream`, for use when reading the received message.

4. Here, we import the Java class `ByteArrayInputStream`, for use when reading the received message.

5. Here we declare a new function that accepts a channel, player, and message. This function serves as the listener function and is called when a message is received. `channel` represents the channel the message was received on. `player` is the player the message was sent through. `message` is a byte array representing the actual payload of the message.

6. Since the message payload is a raw byte array, we need to use the `ByteStreams` class to decode it. Here, we create a new data input to read the message.

7. Here, we read the first part of the message, which is the subchannel of the message, which is the internal channel specified when sending the message.

8. We are interested in receiving the message on our specific internal channel only, so we check if the subchannel is "InternalChannel".

9. Next, we read the length of the inner message payload.

10. Next, we create a new empty byte array, with a length corresponding to the length of the inner message payload we fetched on the previous line.

11. Here, we read the actual content of the inner payload into the empty byte array we created earlier.

12. Next, we initialize a new input stream to translate the inner message payload from a raw byte array to human-readable text.

13. Finally, we read the string value of the inner message payload, which should be the UUID of the player, as specified earlier.

## Summary

- The plugin message manager can be used to send and receive plugin messages.
- To send standard BungeeCord plugin messages, use the `sendBungeeMessage(player, message_type, *payload)` function.
- For a complete list of BungeeCord plugin messages that can be sent/received, see [this page](https://docs.papermc.io/paper/dev/plugin-messaging/).
- To send messages on a custom channel, use the `sendMessage(player, channel, *payload)` function.
- To receive messages, create a listener function and register the listener with the `unregisterListener(listener)` function.
- Plugin message listeners are automatically unregistered when a script is stopped.