# Event Listeners

With PySpigot's ListenerManager, scripts can register event listeners with the underlying platform. When an event fires, its respective function in the script/project will be called, and the event will be passed to the function for the script to use.

For instructions on importing the listener manager into your script, visit the [General Information](../usage.md) page.

???+ info

    This is not a comprehensive guide to events in Bukkit, Velocity, or BungeeCord. For a more complete guide on events, see the respective guide for the platform:
    
    - [Bukkit](https://www.spigotmc.org/wiki/using-the-event-api/)
    - [Velocity](https://docs.papermc.io/velocity/dev/event-api/)
    - [BungeeCord](https://www.spigotmc.org/wiki/event-api/)

## Listener Manager Usage

Available functions depend on the platform:

=== "Bukkit"

    - `registerListener(function, event)`: Registers an event listener. Takes the function to call when the event fires (`function`) as well as the event to listen to (`event`). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `registerListener(function, event, priority)`: Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event). The priority is a Bukkit `EventPriority`. Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - For a list of priorities, see the [EventPriority class](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/EventPriority.html).
    - `registerListener(function, event, ignore_cancelled)`: Facilitates "ignoring" the event if it has been cancelled at an earlier point in time by another event listener, by passing `True` for the `ignore_cancelled` parameter. The listener in the script will not be called if it is cancelled by another listener beforehand. Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - This will only work with events that are [cancellable](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Cancellable.html). Not all events are cancellable.
    - `registerListener(function, event, priority, ignore_cancelled)`: Registers an event listener that is ignored if cancelled *and* that has a priority (a combination of the previous two functions). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `unregisterListener(event_listener)`: Unregisters an event listener. Takes the `ScriptEventListener` that was returned when the listener was registered.
    - `unregisterListener(function, event)`: Unregisters an event listener. Takes the listener function (`function`), and the event being listened to (`event`).
    - `unregisterListeners(script)`: Unregisters all event listeners beloning to a script/project.

=== "Velocity"

    - `registerAsyncListener(function, event, task_type)`: Registers a new asynchronous event listener. Takes the function to call when the event fires (`function`), the event to listen to (`event`), and the type of asynchronous event (`task_type`). For more information, see the section below on [Velocity Asynchronous Listeners](#velocity-asynchronous-listeners). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `registerAsyncListener(function, event, priority, task_type)`: Registers a new asynchronous event listener. Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event).  For more information, see the section below on [Velocity Asynchronous Listeners](#velocity-asynchronous-listeners). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - `priority` is a Java `short`: the minimum value is -32,768, and the maximum value is 32,767. Higher number indicates higher priority (the listener is called *after* listeners with lower priority).
    - `registerListener(function, event)`: Registers an event listener. Takes the function to call when the event fires (`function`) as well as the event to listen to (`event`). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `registerListener(function, event, priority)`: Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - `priority` is a Java `short`: the minimum value is -32,768, and the maximum value is 32,767. Higher number indicates higher priority (the listener is called *after* listeners with lower priority).
    - `unregisterListener(event_listener)`: Unregisters an event listener. Takes the `ScriptEventListener` that was returned when the listener was registered.
    - `unregisterListener(function, event)`: Unregisters an event listener. Takes the listener function (`function`), and the event being listened to (`event`).
    - `unregisterListeners(script)`: Unregisters all event listeners beloning to a script/project.

=== "BungeeCord"

    - `registerListener(function, event)`: Registers an event listener. Takes the function to call when the event fires (`function`) as well as the event to listen to (`event`). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `registerListener(function, event, priority)`: Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - `priority` is a Java `byte`: the minimum value is -128, and the maximum value is 127. Higher number indicates higher priority (the listener is called *after* listeners with lower priority).
    - `unregisterListener(event_listener)`: Unregisters an event listener. Takes the `ScriptEventListener` that was returned when the listener was registered.
    - `unregisterListener(function, event)`: Unregisters an event listener. Takes the listener function (`function`), and the event being listened to (`event`).
    - `unregisterListeners(script)`: Unregisters all event listeners beloning to a script/project.

???+ tip

    You **do not** need to unregister your event listeners when your script is stopped/unloaded. PySpigot will handle this for you.

For a complete list of possible events to listen to, see the respective JavaDocs for the platform:

- [Bukkit events](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/package-summary.html)
- [Velocity events](https://jd.papermc.io/velocity/3.4.0/com/velocitypowered/api/event/package-summary.html)
- [BungeeCord events](https://javadoc.io/doc/net.md-5/bungeecord-api/latest/net/md_5/bungee/api/event/package-summary.html)

## Code Example

Let's look at the following code that defines and registers an event listener:

=== "Bukkit"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from org.bukkit.event.player import AsyncPlayerChatEvent # (2)!

    def player_chat(event): # (3)!
        print('Player sent a chat! Their message was: ' + event.getMessage())

    listener = ps.listener.registerListener(player_chat, AsyncPlayerChatEvent) # (4)!
    ```

    1. Here, we import PySpigot as `ps` to utilize the listener manager (`listener`).

    2. Here, we import Bukkit's `AsyncPlayerChatEvent`, which is the event that we will be listening for. This event is called when a player sends a message to the server. Note that when registering an event listener, the platform's event class *must* be imported!

    3. Here, we define a function called `player_chat` that takes an event as a parameter (an `AsyncPlayerChatEvent` in this case). This is the function that will be called when an AsyncPlayerChatEvent occurs. Within this function, we print a message to the console that contains the message that was sent in chat (the event that fired).

    4. Here, we use the `registerEvent(function, event)` function to register our event listener, passing the function defined earlier, as well as the event we are listening for (`AsyncPlayerChatEvent`). We also assign the returned value of `registerEvent` to `listener`. This is a `ScriptEventListener` object, which represents the listener that was registered. This can be used to unregister the listener if you would like to do so later.

=== "Velocity"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from com.velocitypowered.api.event.player import PlayerChatEvent # (2)!

    def player_chat(event): # (3)!
        print('Player sent a chat! Their message was: ' + event.getMessage())

    listener = ps.listener.registerListener(player_chat, PlayerChatEvent) # (4)!
    ```

    1. Here, we import PySpigot as `ps` to utilize the listener manager (`listener`).

    2. Here, we import Velocity's `PlayerChatEvent`, which is the event that we will be listening for. This event is called when a player sends a message to a server. Note that when registering an event listener, the platform's event class *must* be imported!

    3. Here, we define a function called `player_chat` that takes an event as a parameter (a `PlayerChatEvent` in this case). This is the function that will be called when a PlayerChatEvent occurs. Within this function, we print a message to the console that contains the message that was sent in chat (the event that fired).

    4. Here, we use the `registerEvent(function, event)` function to register our event listener, passing the function defined earlier, as well as the event we are listening for (`PlayerChatEvent`). We also assign the returned value of `registerEvent` to `listener`. This is a `ScriptEventListener` object, which represents the listener that was registered. This can be used to unregister the listener if you would like to do so later.

=== "BungeeCord"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from net.md_5.bungee.api.event import ChatEvent # (2)!

    def player_chat(event): # (3)!
        print('Player sent a chat! Their message was: ' + event.getMessage())

    listener = ps.listener.registerListener(player_chat, ChatEvent) # (4)!
    ```

    1. Here, we import PySpigot as `ps` to utilize the listener manager (`listener`).

    2. Here, we import BungeeCord's `ChatEvent`, which is the event that we will be listening for. This event is called when a player sends a message to a server. Note that when registering an event listener, the platform's event class *must* be imported!

    3. Here, we define a function called `player_chat` that takes an event as a parameter (a `ChatEvent` in this case). This is the function that will be called when a ChatEvent occurs. Within this function, we print a message to the console that contains the message that was sent in chat (the event that fired).

    4. Here, we use the `registerEvent(function, event)` function to register our event listener, passing the function defined earlier, as well as the event we are listening for (`ChatEvent`). We also assign the returned value of `registerEvent` to `listener`. This is a `ScriptEventListener` object, which represents the listener that was registered. This can be used to unregister the listener if you would like to do so later.

All event listeners must be registered with PySpigot's listener manager. In this case, we use the `registerEvent(function, event)` function. It takes two arguments:

- The first argument accepts the function that should be called when the event fires.
- The second argument accepts the platform's event class, the event that should be listened for.

### Unregistering a Listener

Continuing the above code example:

``` py linenums="1"
ps.listener.unregisterListener(listener) # (1)!
```

1. Here, we unregister the listener by passing the `ScriptEventListener` object we assigned earlier when registering the listener.

## Velocity Asynchronous Listeners

As of Velocity 3.0.0, events can be handled asynchronously. The event system allows an event listener to "pause" sending an event to every listener, perform some sort of additional computation or I/O work, and then "resume" processing the event after the work is finished. The additional work to be completed is referred to as a "task". There are three types of tasks that may be attached to asynchronous listeners, and each function somewhat differently:

### Basic Asynchronous Tasks

Asynchronous tasks simply run a unit of execution asynchronously. Basic asynchronous tasks are the closest equivalent for Velocity 1.x.x style event listeners and asynchronous events in the Bukkit API. Event processing is *not* paused with the asynchronous task type.

To register an asynchronous listener with an asynchronous task type, pass `EventTaskType.ASYNC` for the `task_type` parameter when registering the listener:

``` py linenums="1"
import pyspigot as ps
from dev.magicmq.pyspigot.velocity.manager.listener import EventTaskType # (1)!
from com.velocitypowered.api.event.player import PlayerChatEvent

def player_chat(event):
    print('Player sent a chat! Their message was: ' + event.getMessage())

listener = ps.listener.registerAsyncListener(player_chat, PlayerChatEvent, EventTaskType.ASYNC) # (2)!
```

1. Here, we import `EventTaskType` so that it can be specified when the event listener is registered.

2. Here, we register the event like before but this time by using `registerAsyncListener` and by passing the `EventTaskType` we wish to use.

### Continuation Tasks

The continuation task provides the listener with a callback (known as a `Continuation`) to resume event processing when the work is completed. Continuation-based event listeners are the closest equivalent for listeners that use BungeeCord `AsyncEvent` intents. Note that continuation listeners pass an additional parameter to the event listener function, the continuation. The continuation **must** be notified from the event listener function when the additional work is completed.

To register an asynchronous listener with a continuation task type, pass `EventTaskType.CONTINUATION` for the `task_type` parameter when registering the listener:

``` py linenums="1"
import pyspigot as ps
from dev.magicmq.pyspigot.velocity.manager.listener import EventTaskType # (1)!
from com.velocitypowered.api.event.player import PlayerChatEvent

def player_chat(event, continuation): # (2)!
    print('Player sent a chat! Their message was: ' + event.getMessage())
    # Do some work here...
    continuation.resume() # (3)!

listener = ps.listener.registerAsyncListener(player_chat, PlayerChatEvent, EventTaskType.ASYNC) # (4)!
```

1. Here, we import `EventTaskType` so that it can be specified when the event listener is registered.

2. The listener function accepts an additional parameter `continuation` so that it can be notified when the work completes.

3. Once the work is completed, the continuation is notified via `continuation.resume()`. Note that when using a continuation task type, the continuation **must** be notified, or the event will stall (and it won't proceed to the next listener). If there was an exception that occurred when doing the work, the continuation can be notified via `continuation.resumeWithException(exception)`, passing the exception that occurred.

4. Here, we register the event like before but this time by using `registerAsyncListener` and by passing the `EventTaskType` we wish to use.

### Resume When Complete Tasks

The resume when complete task type is similar to a continuation task, except that event processing is automatically continued when the script/project's event listener finishes execution. With this event task type, there is no need to notify the continuation manually when the additional work is completed.

To register an asynchronous listener with a resume when complete task type, pass `EventTaskType.RESUME_WHEN_COMPLETE` for the `task_type` parameter when registering the listener:

``` py linenums="1"
import pyspigot as ps
from dev.magicmq.pyspigot.velocity.manager.listener import EventTaskType # (1)!
from com.velocitypowered.api.event.player import PlayerChatEvent

def player_chat(event):
    print('Player sent a chat! Their message was: ' + event.getMessage())
    # Do some work here...

listener = ps.listener.registerAsyncListener(player_chat, PlayerChatEvent, EventTaskType.RESUME_WHEN_COMPLETE) # (2)!
```

1. Here, we import `EventTaskType` so that it can be specified when the event listener is registered.

2. Here, we register the event like before but this time by using `registerAsyncListener` and by passing the `EventTaskType` we wish to use.

## Summary

- All events that you wish to use should be imported using Python's import syntax.
- All event listeners should be defined as functions in your script that accept a single parameter, the event (the parameter name can be whatever you like).
- All event listeners must be registered with PySpigot's listener manager. The most basic function to register an event listener is `listener.registerEvent(function, event)`.
- When registering an event listener, the `registerEvent` functions all return a `ScriptEventListener`, which can be used to unregister the listener.
