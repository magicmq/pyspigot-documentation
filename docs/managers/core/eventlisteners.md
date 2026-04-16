# Event Listeners

With PySpigot's ListenerManager, scripts can register event listeners with the underlying platform. When an event fires, its respective function in the script/project will be called, and the event will be passed to the function for the script to use.

For instructions on importing the listener manager into your script, visit the [General Information](../usage.md) page.

???+ info

    This is not a comprehensive guide to events in Bukkit, Velocity, or BungeeCord. For a more complete guide on events, see the respective guide for the platform:
    
    - [Bukkit](https://www.spigotmc.org/wiki/using-the-event-api/)
    - [Velocity](https://docs.papermc.io/velocity/dev/event-api/)
    - [BungeeCord](https://www.spigotmc.org/wiki/event-api/)

## The `event_listener` Decorator

PySpigot ships with a `decorators/event_listener.py` helper module that provides a Python **decorator** for registering event listeners. Using the decorator is the recommended way to register listeners, as it is cleaner and more Pythonic than calling the listener manager directly.

### Importing

Import the decorator from the module:

=== "Bukkit"

    ``` py
    from decorators.event_listener import event_listener
    ```

=== "Velocity"

    ``` py
    from decorators.event_listener import event_listener
    from decorators.event_listener import async_event_listener  # (1)!
    ```

    1. `async_event_listener` is only needed if you want to register an [asynchronous event listener](#velocity-asynchronous-listeners).

=== "BungeeCord"

    ``` py
    from decorators.event_listener import event_listener
    ```

### Basic Usage

Apply `@event_listener(EventClass)` to a function to register it as a listener for that event:

=== "Bukkit"

    ``` py linenums="1"
    from decorators.event_listener import event_listener
    from org.bukkit.event.player import AsyncPlayerChatEvent # (1)!

    @event_listener(AsyncPlayerChatEvent) # (2)!
    def player_chat(event): # (3)!
        print('Player sent a chat! Their message was: ' + event.getMessage())
    ```

    1. The event class *must* be imported before it is passed to the decorator.

    2. Applying `@event_listener(AsyncPlayerChatEvent)` registers `player_chat` as a listener for `AsyncPlayerChatEvent`. The decorator wraps the function and registers it automatically — no separate call to the listener manager is needed.

    3. The function receives the event as its only parameter. It can be named anything you like.

=== "Velocity"

    ``` py linenums="1"
    from decorators.event_listener import event_listener
    from com.velocitypowered.api.event.player import PlayerChatEvent # (1)!

    @event_listener(PlayerChatEvent) # (2)!
    def player_chat(event): # (3)!
        print('Player sent a chat! Their message was: ' + event.getMessage())
    ```

    1. The event class *must* be imported before it is passed to the decorator.

    2. Applying `@event_listener(PlayerChatEvent)` registers `player_chat` as a listener for `PlayerChatEvent`. The decorator wraps the function and registers it automatically — no separate call to the listener manager is needed.

    3. The function receives the event as its only parameter. It can be named anything you like.

=== "BungeeCord"

    ``` py linenums="1"
    from decorators.event_listener import event_listener
    from net.md_5.bungee.api.event import ChatEvent # (1)!

    @event_listener(ChatEvent) # (2)!
    def player_chat(event): # (3)!
        print('Player sent a chat! Their message was: ' + event.getMessage())
    ```

    1. The event class *must* be imported before it is passed to the decorator.

    2. Applying `@event_listener(ChatEvent)` registers `player_chat` as a listener for `ChatEvent`. The decorator wraps the function and registers it automatically — no separate call to the listener manager is needed.

    3. The function receives the event as its only parameter. It can be named anything you like.

### Optional Parameters

The `event_listener` decorator accepts optional parameters to customize listener behavior:

=== "Bukkit"

    - `priority` — controls how early or late the listener fires relative to other listeners for the same event. Accepts a Bukkit `EventPriority`. Defaults to `EventPriority.NORMAL`.
        - For a list of priorities, see the [EventPriority class](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/EventPriority.html).
    - `ignore_cancelled` — when `True`, the listener will not be called if the event has already been cancelled by an earlier listener. Defaults to `False`.
        - This only applies to events that implement [Cancellable](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Cancellable.html). Not all events are cancellable.

    ``` py linenums="1"
    from decorators.event_listener import event_listener
    from org.bukkit.event import EventPriority
    from org.bukkit.event.player import AsyncPlayerChatEvent

    @event_listener(AsyncPlayerChatEvent, priority=EventPriority.HIGH, ignore_cancelled=True) # (1)!
    def player_chat(event):
        print('Player sent a chat! Their message was: ' + event.getMessage())
    ```

    1. This listener fires at `HIGH` priority and will be skipped if the event has been cancelled by a higher-priority listener.

=== "Velocity"

    - `priority` — controls how early or late the listener fires relative to other listeners for the same event. Accepts a Java `short` value. Higher numbers indicate higher priority (the listener is called *after* listeners with lower priority). Defaults to `0`.
        - The minimum value is -32,768 and the maximum value is 32,767.

    ``` py linenums="1"
    from decorators.event_listener import event_listener
    from com.velocitypowered.api.event.player import PlayerChatEvent

    @event_listener(PlayerChatEvent, priority=100) # (1)!
    def player_chat(event):
        print('Player sent a chat! Their message was: ' + event.getMessage())
    ```

    1. This listener fires at priority `100`, after listeners registered with a lower priority.

=== "BungeeCord"

    - `priority` — controls how early or late the listener fires relative to other listeners for the same event. Accepts a Java `byte` value. Higher numbers indicate higher priority (the listener is called *after* listeners with lower priority). Defaults to `EventPriority.NORMAL`.
        - The minimum value is -128 and the maximum value is 127.

    ``` py linenums="1"
    from decorators.event_listener import event_listener
    from net.md_5.bungee.api.event import ChatEvent

    @event_listener(ChatEvent, priority=50) # (1)!
    def player_chat(event):
        print('Player sent a chat! Their message was: ' + event.getMessage())
    ```

    1. This listener fires at priority `50`, after listeners registered with a lower priority.

### Unregistering a Listener

When a function is decorated with `@event_listener`, two attributes are attached to it:

- `.registered_listener` — the `ScriptEventListener` object representing the registered listener.
- `.unregister()` — a convenience method that unregisters the listener.

``` py linenums="1"
player_chat.unregister() # (1)!
```

1. Unregisters the `player_chat` listener. After this call, the function will no longer be called when the event fires.

???+ tip

    You **do not** need to unregister your event listeners when your script is stopped/unloaded. PySpigot will handle this for you.

For a complete list of possible events to listen to, see the respective JavaDocs for the platform:

- [Bukkit events](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/package-summary.html)
- [Velocity events](https://jd.papermc.io/velocity/3.4.0/com/velocitypowered/api/event/package-summary.html)
- [BungeeCord events](https://javadoc.io/doc/net.md-5/bungeecord-api/latest/net/md_5/bungee/api/event/package-summary.html)

## Velocity Asynchronous Listeners

As of Velocity 3.0.0, events can be handled asynchronously. The event system allows an event listener to "pause" sending an event to every listener, perform some sort of additional computation or I/O work, and then "resume" processing the event after the work is finished. The additional work to be completed is referred to as a "task". There are three types of tasks that may be attached to asynchronous listeners, and each functions somewhat differently.

Use the `async_event_listener` decorator (imported from `decorators.event_listener`) to register an asynchronous listener. It accepts the following parameters:

- `event_class` — the event to listen to.
- `event_task_type` — the type of asynchronous task. See the sections below for details on each type.
- `priority` — how early or late the listener fires relative to others (Java `short`, defaults to `0`).

### Basic Asynchronous Tasks

Asynchronous tasks simply run a unit of execution asynchronously. Basic asynchronous tasks are the closest equivalent for Velocity 1.x.x style event listeners and asynchronous events in the Bukkit API. Event processing is *not* paused with the asynchronous task type.

Use `EventTaskType.ASYNC` for the `event_task_type` parameter:

``` py linenums="1"
from decorators.event_listener import async_event_listener
from dev.magicmq.pyspigot.velocity.manager.listener import EventTaskType # (1)!
from com.velocitypowered.api.event.player import PlayerChatEvent

@async_event_listener(PlayerChatEvent, EventTaskType.ASYNC) # (2)!
def player_chat(event):
    print('Player sent a chat! Their message was: ' + event.getMessage())
```

1. Import `EventTaskType` so that it can be specified when the event listener is registered.

2. Decorating with `@async_event_listener(PlayerChatEvent, EventTaskType.ASYNC)` registers `player_chat` as an asynchronous listener for `PlayerChatEvent`.

### Continuation Tasks

The continuation task provides the listener with a callback (known as a `Continuation`) to resume event processing when the work is completed. Continuation-based event listeners are the closest equivalent for listeners that use BungeeCord `AsyncEvent` intents. Note that continuation listeners pass an additional parameter to the event listener function — the continuation. The continuation **must** be notified from the event listener function when the additional work is completed.

Use `EventTaskType.CONTINUATION` for the `event_task_type` parameter:

``` py linenums="1"
from decorators.event_listener import async_event_listener
from dev.magicmq.pyspigot.velocity.manager.listener import EventTaskType # (1)!
from com.velocitypowered.api.event.player import PlayerChatEvent

@async_event_listener(PlayerChatEvent, EventTaskType.CONTINUATION) # (2)!
def player_chat(event, continuation): # (3)!
    print('Player sent a chat! Their message was: ' + event.getMessage())
    # Do some work here...
    continuation.resume() # (4)!
```

1. Import `EventTaskType` so that it can be specified when the event listener is registered.

2. Decorating with `@async_event_listener(PlayerChatEvent, EventTaskType.CONTINUATION)` registers `player_chat` as a continuation-based listener.

3. The listener function accepts an additional parameter `continuation` so that it can be notified when the work completes.

4. Once the work is completed, the continuation is notified via `continuation.resume()`. The continuation **must** be notified, or event processing will stall and the event won't proceed to the next listener. If an exception occurred, notify via `continuation.resumeWithException(exception)`, passing the exception.

### Resume When Complete Tasks

The resume when complete task type is similar to a continuation task, except that event processing is automatically continued when the listener function finishes execution. With this event task type, there is no need to notify the continuation manually.

Use `EventTaskType.RESUME_WHEN_COMPLETE` for the `event_task_type` parameter:

``` py linenums="1"
from decorators.event_listener import async_event_listener
from dev.magicmq.pyspigot.velocity.manager.listener import EventTaskType # (1)!
from com.velocitypowered.api.event.player import PlayerChatEvent

@async_event_listener(PlayerChatEvent, EventTaskType.RESUME_WHEN_COMPLETE) # (2)!
def player_chat(event):
    print('Player sent a chat! Their message was: ' + event.getMessage())
    # Do some work here...
```

1. Import `EventTaskType` so that it can be specified when the event listener is registered.

2. Decorating with `@async_event_listener(PlayerChatEvent, EventTaskType.RESUME_WHEN_COMPLETE)` registers `player_chat` as a resume-when-complete listener. Event processing continues automatically once `player_chat` returns.

## Listener Manager Usage

If you prefer to register listeners manually without using the decorator, the listener manager functions are available as an alternative. The functions available depend on the platform:

=== "Bukkit"

    - `registerListener(function, event)`: Registers an event listener. Takes the function to call when the event fires (`function`) as well as the event to listen to (`event`). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `registerListener(function, event, priority)`: Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event). The priority is a Bukkit `EventPriority`. Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - For a list of priorities, see the [EventPriority class](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/EventPriority.html).
    - `registerListener(function, event, ignore_cancelled)`: Facilitates "ignoring" the event if it has been cancelled at an earlier point in time by another event listener, by passing `True` for the `ignore_cancelled` parameter. The listener in the script will not be called if it is cancelled by another listener beforehand. Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - This will only work with events that are [cancellable](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Cancellable.html). Not all events are cancellable.
    - `registerListener(function, event, priority, ignore_cancelled)`: Registers an event listener that is ignored if cancelled *and* that has a priority (a combination of the previous two functions). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `unregisterListener(event_listener)`: Unregisters an event listener. Takes the `ScriptEventListener` that was returned when the listener was registered.
    - `unregisterListener(function, event)`: Unregisters an event listener. Takes the listener function (`function`), and the event being listened to (`event`).
    - `unregisterListeners(script)`: Unregisters all event listeners belonging to a script/project.

=== "Velocity"

    - `registerAsyncListener(function, event, task_type)`: Registers a new asynchronous event listener. Takes the function to call when the event fires (`function`), the event to listen to (`event`), and the type of asynchronous event (`task_type`). For more information, see the section above on [Velocity Asynchronous Listeners](#velocity-asynchronous-listeners). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `registerAsyncListener(function, event, priority, task_type)`: Registers a new asynchronous event listener. Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event). For more information, see the section above on [Velocity Asynchronous Listeners](#velocity-asynchronous-listeners). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - `priority` is a Java `short`: the minimum value is -32,768, and the maximum value is 32,767. Higher number indicates higher priority (the listener is called *after* listeners with lower priority).
    - `registerListener(function, event)`: Registers an event listener. Takes the function to call when the event fires (`function`) as well as the event to listen to (`event`). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `registerListener(function, event, priority)`: Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - `priority` is a Java `short`: the minimum value is -32,768, and the maximum value is 32,767. Higher number indicates higher priority (the listener is called *after* listeners with lower priority).
    - `unregisterListener(event_listener)`: Unregisters an event listener. Takes the `ScriptEventListener` that was returned when the listener was registered.
    - `unregisterListener(function, event)`: Unregisters an event listener. Takes the listener function (`function`), and the event being listened to (`event`).
    - `unregisterListeners(script)`: Unregisters all event listeners belonging to a script/project.

=== "BungeeCord"

    - `registerListener(function, event)`: Registers an event listener. Takes the function to call when the event fires (`function`) as well as the event to listen to (`event`). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
    - `registerListener(function, event, priority)`: Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event). Returns a `ScriptEventListener`, which can be used to unregister the event listener later.
        - `priority` is a Java `byte`: the minimum value is -128, and the maximum value is 127. Higher number indicates higher priority (the listener is called *after* listeners with lower priority).
    - `unregisterListener(event_listener)`: Unregisters an event listener. Takes the `ScriptEventListener` that was returned when the listener was registered.
    - `unregisterListener(function, event)`: Unregisters an event listener. Takes the listener function (`function`), and the event being listened to (`event`).
    - `unregisterListeners(script)`: Unregisters all event listeners belonging to a script/project.

### Code Example

The following shows how to register and unregister a listener using the listener manager directly:

=== "Bukkit"

    ``` py linenums="1"
    import pyspigot as ps
    from org.bukkit.event.player import AsyncPlayerChatEvent

    def player_chat(event):
        print('Player sent a chat! Their message was: ' + event.getMessage())

    listener = ps.listener_manager().registerListener(player_chat, AsyncPlayerChatEvent) # (1)!

    # Later, to unregister:
    ps.listener_manager().unregisterListener(listener) # (2)!
    ```

    1. Register the listener manually via the listener manager. The returned `ScriptEventListener` is saved so it can be used to unregister later.

    2. Unregister the listener by passing the `ScriptEventListener` returned during registration.

=== "Velocity"

    ``` py linenums="1"
    import pyspigot as ps
    from com.velocitypowered.api.event.player import PlayerChatEvent

    def player_chat(event):
        print('Player sent a chat! Their message was: ' + event.getMessage())

    listener = ps.listener_manager().registerListener(player_chat, PlayerChatEvent) # (1)!

    # Later, to unregister:
    ps.listener_manager().unregisterListener(listener) # (2)!
    ```

    1. Register the listener manually via the listener manager. The returned `ScriptEventListener` is saved so it can be used to unregister later.

    2. Unregister the listener by passing the `ScriptEventListener` returned during registration.

=== "BungeeCord"

    ``` py linenums="1"
    import pyspigot as ps
    from net.md_5.bungee.api.event import ChatEvent

    def player_chat(event):
        print('Player sent a chat! Their message was: ' + event.getMessage())

    listener = ps.listener_manager().registerListener(player_chat, ChatEvent) # (1)!

    # Later, to unregister:
    ps.listener_manager().unregisterListener(listener) # (2)!
    ```

    1. Register the listener manually via the listener manager. The returned `ScriptEventListener` is saved so it can be used to unregister later.

    2. Unregister the listener by passing the `ScriptEventListener` returned during registration.

## Summary

- All events that you wish to use should be imported using Python's import syntax.
- All event listeners should be defined as functions in your script that accept a single parameter, the event (the parameter name can be whatever you like).
- The recommended way to register a listener is via the `@event_listener(EventClass)` decorator from `decorators/event_listener.py`.
- Decorated functions gain a `.registered_listener` attribute (the `ScriptEventListener`) and an `.unregister()` method for easy cleanup.
- Alternatively, listeners can be registered manually via the listener manager using `registerListener(function, event)` and related functions.
- You **do not** need to unregister listeners when your script stops — PySpigot handles cleanup automatically.
