# Event Listeners

With PySpigot's ListenerManager, scripts can register event listeners. Like Bukkit's listener system, when an event fires, its respective function in the script will be called, and the event can be utilized within the script.

For instructions on importing the listener manager into your script, visit the [General Information](usage.md) page.

???+ info

    This is not a comprehensive guide to events in Bukkit. For a more complete guide on events, see [Spigot's tutorial](https://www.spigotmc.org/wiki/using-the-event-api/) on using the event API.

## Listener Manager Usage

There are five functions available to use from the listener manager:

- `registerListener(function, event)`: Registers an event listener. Takes the function to call when the event fires (`function`) as well as the event to listen to (`event`). Returns a `ScriptEventListener`, which can be used to unregister the event later.
- `registerListener(function, event, priority)`: Same as above, except also takes a `priority` (how early/late the event listener should fire relative to other listeners for the same event). The priority is a string, and represents an `EventPriority`. Returns a `ScriptEventListener`, which can be used to unregister the event later.
    - Event priorities are the same as the priorities found in Bukkit's [EventPriority class](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/EventPriority.html).
- `registerListener(function, event, ignoreCancelled)`: Facilitates "ignoring" the event if it has been cancelled at an earlier point in time by another event listener. The listener in the script will not be called if it is cancelled elsewhere beforehand. Returns a `ScriptEventListener`, which can be used to unregister the event later.
    - This will only work with events that are [cancellable](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Cancellable.html). Not all events are cancellable.
- `registerListener(function, event, priority, ignoreCancelled)`: Registers an event listener that is ignored if cancelled *and* that has a priority (a combination of the previous two functions). Returns a `ScriptEventListener`, which can be used to unregister the event later.
- `unregisterListener(event_listener)`: Unregisters an event listener. Takes the `ScriptEventListener` that was returned when the listener was registered.

???+ tip

    You **do not** need to unregister your event listeners when your script is stopped/unloaded. PySpigot will handle this for you.

## Code Example

Let's look at the following code that defines and registers an event listener:

``` py linenums="1"
import pyspigot as ps # (1)!
from org.bukkit.event.player import AsyncPlayerChatEvent # (2)!

def player_chat(event): # (3)!
    print('Player sent a chat! Their message was: ' + event.getMessage())

listener = ps.listener.registerListener(player_chat, AsyncPlayerChatEvent) # (4)!
```

1. Here, we import PySpigot as `ps` to utilize the listener manager (`listener`).

2. Here, we import Bukkit's `AsyncPlayerChatEvent`, which is the event that we will be listening for. All events that you wish to listen to *must* be imported!

3. Here, we define a function called `player_chat` that takes an event as a parameter (an AsyncPlayerChatEvent in this case). This is the function that will be called when an AsyncPlayerChatEvent occurs. Within this function, we print a message to the console that contains the message that was sent in chat (the event that fired).

4. Here, we use the `registerEvent(function, event)` function to register our event listener, passing the function defined earlier, as well as the event we are listening for (`AsyncPlayerChatEvent`). We also assign the returned value of `registerEvent` to `listener`. This is a `ScriptEventListener` object, which represents the listener that was registered. This can be used to unregister the listener if you would like to do so later.

All event listeners must be registered with PySpigot's listener manager. In this case, we use the `registerEvent(function, event)` function. It takes two arguments:

- The first argument accepts the function that should be called when the event fires.
- The second argument accepts the event that should be listened for.

### Unregistering a Listener

Continuing the above code example:

``` py linenums="1"
ps.listener.unregisterListener(listener) # (1)!
```

1. Here, we unregister the listener by passing the `ScriptEventListener` object we assigned earlier when registering the listener.

For complete documentation on available listeners and functions/methods available to use from each, see the [Spigot JavaDocs](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Event.html).

## Summary

- All events that you wish to use should be imported using Python's import syntax.
- All event listeners should be defined as functions in your script that accept a single parameter, the event (the parameter name can be whatever you like).
- All event listeners must be registered with PySpigot's listener manager using `listener.registerEvent(function, event)`.
- When registering an event listener, the `registerEvent` functions all return a `ScriptEventListener`, which can be used to unregister the listener.
