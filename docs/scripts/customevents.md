# Custom Script Events

PySpigot includes a custom event, called `CustomEvent`, which can be used to create and call a custom event from within a script when something occurs. This event is designed for scripts to signal such an event to other scripts or plugins. 

The event includes a `data` variable, set when the event is created, that allows the script to pass any type of data to listeners that are listening to the event. Also included is a `name` variable, which allows for greater specificity for listeners when multiple scripts are utilizing the `CustomEvent` simultaneously.

PySpigot also automatically attaches the script that created and called the event to the event itself, so that listeners can determine which script called the event.

The following describes how to use this custom event.

## Utilizing the Custom Event

### Importing the Custom Event

Accessing the custom event class depends on the platform:

=== "Bukkit"

    On Bukkit, the `CustomEvent` lives at `dev.magicmq.pyspigot.bukkit.event.custom.CustomEvent`:

    ``` py linenums="1"
    from dev.magicmq.pyspigot.bukkit.event.custom import CustomEvent
    ```

=== "Velocity"

    On Velocity, the `CustomEvent` lives at `dev.magicmq.pyspigot.velocity.event.custom.CustomEvent`:

    ``` py linenums="1"
    from dev.magicmq.pyspigot.velocity.event.custom import CustomEvent
    ```

=== "BungeeCord"

    On BungeeCord, the `CustomEvent` lives at `dev.magicmq.pyspigot.bungee.event.custom.CustomEvent`:

    ``` py linenums="1"
    from dev.magicmq.pyspigot.bungee.event.custom import CustomEvent
    ```

### Creating the Custom Event

When a new `CustomEvent` object instance is created, it can accept up to three parameters (the first two are required, the third is optional):

- `name`: The name of the event. Usually, this should be a specific name that an event listener can use to differentiate between multiple `CustomEvent` events coming from different scripts.
- `data`: The data attached to the event. The code example below creates some dummy data for this, but typically this will be some useful data related to the event that an event listener can utilize.
    - *Any* Python type can be used for the `data` parameter; no type conversion is required.
- `async` (optional, Bukkit platform only): If the event is being created and called from an asynchronous context (such as from within an asynchronous task), this should be set to `True`. Otherwise, it can be ommitted or set to `False`.
    - *Note*: On the Velocity and BungeeCord platforms, there is no `async` parameter, because *all* events are considered asynchronous on these platforms.

=== "Bukkit"

    ``` py linenums="1"
    from dev.magicmq.pyspigot.bukkit.event.custom import CustomEvent

    dictionary = {'test': '1', 'test2': '2'}

    # Create the custom event with the dictionary as the data
    event = CustomEvent('script1_event', dictionary)
    ```

    If the event is being called from an asynchronous context, the event should be created like so:

    ``` py linenums="1"
    from dev.magicmq.pyspigot.event.custom import CustomEvent

    dictionary = {'test': '1', 'test2': '2'}

    # Create the custom event with the dictionary as the data
    event = CustomEvent('script1_event', dictionary, True)
    ```

=== "Velocity"

    ``` py linenums="1"
    from dev.magicmq.pyspigot.velocity.event.custom import CustomEvent

    dictionary = {'test': '1', 'test2': '2'}

    # Create the custom event with the dictionary as the data
    event = CustomEvent('script1_event', dictionary)
    ```

    Note that *all* events should be considered asynchronous on Velocity.

=== "BungeeCord"

    ``` py linenums="1"
    from dev.magicmq.pyspigot.bungee.event.custom import CustomEvent

    dictionary = {'test': '1', 'test2': '2'}

    # Create the custom event with the dictionary as the data
    event = CustomEvent('script1_event', dictionary)
    ```

    Note that *all* events should be considered asynchronous on BungeeCord.

### Calling the Custom Event

The custom event is called from the platform-specific plugin/event manager:

=== "Bukkit"

    On Bukkit, the plugin manager is accessible via the `Bukkit` class (`org.bukkit.Bukkit`):

    ``` py linenums="1"
    from org.bukkit import Bukkit

    ...

    Bukkit.getPluginManager().callEvent(event)
    ```

=== "Velocity"

    On Velocity, the event manager is accessible via the main plugin class, `PyVelocity` (`dev.magicmq.pyspigot.velocity.PyVelocity`):

    ``` py linenums="1"
    from dev.magicmq.pyspigot.velocity import PyVelocity

    ...

    PyVelocity.get().getProxy().getEventManager().fireAndForget(event)
    ```

    Velocity's event manager also includes a method called `fire`, which can be used to fetch the event object after it has been processed by listeners, either asynchronously or by waiting until processing completes:

    ``` py linenums="1"
    from dev.magicmq.pyspigot.velocity import PyVelocity

    ...

    future = PyVelocity.get().getProxy().getEventManager().fire(event) # (1)!

    processed_event = future.get() # (2)!
    ```

    1.  `fire` returns a `CompletableFuture`, which is an object that represents work that may be completed at some point in the future. For more information, see [this page](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html).
    2.  The `get` method waits (blocks the main thread) until the work finishes (I.E. all event listeners process the event).

=== "BungeeCord"

    On BungeeCord, the plugin manager is accessible via the `ProxyServer` class (`net.md_5.bungee.api.ProxyServer`):

    ``` py linenums="1"
    from net.md_5.bungee.api import ProxyServer

    ...

    ProxyServer.getInstance().getPluginManager().callEvent(event)
    ```

## Code Example

Putting it all together, here is a functional example:

=== "Bukkit"

    ``` py linenums="1"
    from dev.magicmq.pyspigot.bukkit.event.custom import CustomEvent # (1)!
    from org.bukkit import Bukkit # (2)!

    dictionary = {'test': '1', 'test2': '2'} # (3)!

    event = CustomEvent('script1_event', dictionary) # (4)!

    Bukkit.getServer().getPluginManager().callEvent(event) # (5)!
    ```

    1. Here, we import the `CustomEvent` class.

    2. Here, we import the `Bukkit` class.

    3. Here, we create a dummy dictionary object that will serve as our data for the event.

    4. Here, we create a new instance of the `CustomEvent`, and pass it a specific name ("script1_event" in this case), as well as the dictionary data we created earlier.

    5. Here, we call the event with Bukkit's plugin manager, by passing the `CustomEvent` object we created earlier to `callEvent()`.

=== "Velocity"

    ``` py linenums="1"
    from dev.magicmq.pyspigot.velocity.event.custom import CustomEvent # (1)!
    from dev.magicmq.pyspigot.velocity import PyVelocity # (2)!

    dictionary = {'test': '1', 'test2': '2'} # (3)!

    event = CustomEvent('script1_event', dictionary) # (4)!

    PyVelocity.get().getProxy().getEventManager().fireAndForget(event) # (5)!
    ```

    1. Here, we import the `CustomEvent` class.

    2. Here, we import the `PyVelocity` class.

    3. Here, we create a dummy dictionary object that will serve as our data for the event.

    4. Here, we create a new instance of the `CustomEvent`, and pass it a specific name ("script1_event" in this case), as well as the dictionary data we created earlier.

    5. Here, we call the event with Velocity's event manager, by passing the `CustomEvent` object we created earlier to `fireAndForget()`.

=== "BungeeCord"

    ``` py linenums="1"
    from dev.magicmq.pyspigot.bungee.event.custom import CustomEvent # (1)!
    from net.md_5.bungee.api import ProxyServer # (2)!

    dictionary = {'test': '1', 'test2': '2'} # (3)!

    event = CustomEvent('script1_event', dictionary) # (4)!

    ProxyServer.getInstance().getPluginManager().callEvent(event) # (5)!
    ```

    1. Here, we import the `CustomEvent` class.

    2. Here, we import the `ProxyServer` class.

    3. Here, we create a dummy dictionary object that will serve as our data for the event.

    4. Here, we create a new instance of the `CustomEvent`, and pass it a specific name ("script1_event" in this case), as well as the dictionary data we created earlier.

    5. Here, we call the event with BungeeCord's plugin manager, by passing the `CustomEvent` object we created earlier to `callEvent()`.

## Resulted Events on Velocity

The Velocity platform API includes a special type of event called [`ResultedEvent`](https://jd.papermc.io/velocity/3.4.0/com/velocitypowered/api/event/ResultedEvent.html), which essentially specifies an event that has some type of result or outcome attached to it. PySpigot includes an extension of this event called `CustomResultedEvent`. This event is most similar to the idea of cancellable events in Bukkit; the "result" is whether or not the event is cancelled. It is imported and instances of it are created in the same way as the regular `CustomEvent`:

``` py linenums="1"
from dev.magicmq.pyspigot.velocity.event.custom import CustomResultedEvent

dictionary = {'test': '1', 'test2': '2'}

# Create the custom resulted event with the dictionary as the data
resulted_event = CustomResultedEvent('script1_event', dictionary)
```

Because the event is designed with some sort of outcome in mind, it includes `setResult()` and `getResult()` methods, which set and get a [`GenericResult`](https://jd.papermc.io/velocity/3.4.0/com/velocitypowered/api/event/ResultedEvent.GenericResult.html) object, respectively. The `GenericResult` object can be used to set the allowed/denied status of the event via the `allow()` and `deny()` methods. In this context, calling `allow()` is similar to calling `setCancelled(false)` in Bukkit, and calling `deny()` is similar to calling `setCancelled(true)` in Bukkit.

The `CustomResultedEvent` is called in the same way as a regular `CustomEvent`:

``` py linenums="1"
from dev.magicmq.pyspigot.velocity.event.custom import CustomResultedEvent
from dev.magicmq.pyspigot.velocity import PyVelocity

dictionary = {'test': '1', 'test2': '2'}

event = CustomResultedEvent('script1_event', dictionary)

future = PyVelocity.get().getProxy().getEventManager().fire(event) # (1)!

outcome = future.get() # (2)!

allowed = outcome.getResult().isAllowed() # (3)!
```

1.  `fire` returns a `CompletableFuture`, which is an object that represents work that may be completed at some point in the future. For more information, see [this page](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html).
2.  The `get` method waits (blocks the main thread) until the work finishes and an outcome is available (I.E. all event listeners process the event). The outcome is assigned to the `outcome` variable.

3.  Get the result of the event, and get whether the result is allowed or denied with `isAllowed()`.

???+ note

    Velocity's resulted event system is more robust than the `GenericResult` above; for example, a [`ComponentResult`](https://jd.papermc.io/velocity/3.4.0/com/velocitypowered/api/event/ResultedEvent.ComponentResult.html) also exists, which representes an "allowed/denied" result but also contains a adventure API component which specifies a "reason" for denial. For now, only `GenericResult` is supported.

## Listening to the Custom Event

Listening to the custom event is done in the same way as you would listen to any other Bukkit/Velocity/BungeeCord event. The `CustomEvent` class itself has multiple functions available to use:

- `event.getScript()`: Get the script that created and called this event.
- `event.getName()`: Get the name of the event.
- `event.getData()`: Get the data attached to the event. This is returned as a `PyObject`, which can be used directly from within a script without any type coercion.
- `event.getDataAsType(String clazz)`: Get the data attached to the event as an instance of the provided class name. This method is useful if you can't cast the `PyObject` data to the appropriate Java type.
- `event.getDataAsType(Class<T> clazz)`: Get the data attached to the event as an instance of the provided class. This method is useful if you can't cast the `PyObject` data to the appropriate Java type.

???+ tip

    On Bukkit only, the `CustomEvent` is [cancellable](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Cancellable.html), which means a listener may cancel the event if desired. This feature is not supported on Velocity or BungeeCord.

## Summary

- The `CustomEvent` class allows scripts to signal events to other scripts or plugins, with support for passing custom `data` and specifying an event `name`.
- Creating the `CustomEvent` accepts multiple parameters: `name` (event identifier), `data` (any Python type), and `async` (optional, for asynchronous contexts).
- Call the event by using the platform's plugin/event manager. On Bukkit, this would be done via `Bukkit.getPluginManager().callEvent(event)`.
- Other scripts and plugins can listen for `CustomEvent` in the same way as any other Bukkit/Velocity/BungeeCord event. Key event functions include `getScript()` (returns the calling script), `getName()` (returns the event name), and `getData()` (returns the data attached to the event).