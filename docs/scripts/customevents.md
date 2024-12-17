# Custom Script Events

PySpigot includes a custom event, called `CustomEvent`, which can be used to create and call a custom event from within a script when something occurs. This event is designed for scripts to signal an event to other scripts and plugins when a specific event occurs. 

The event includes a `data` variable, set when the event is created, that allows the script to pass any type of data to listeners that are listening to the event. Also included is a `name` variable, that allows for greater specificity for listeners when multiple scripts are utilizing the `CustomEvent` simultaneously.

PySpigot also automatically attaches the script that created and called the event to the event itself, so that listeners can determine which script called the event.

The following describes how to use this custom event.

## Utilizing the Custom Event

### Importing the Custom Event

The `CustomEvent` class is accessible via `dev.magicmq.pyspigot.event.custom.CustomEvent`:

``` py linenums="1"
from dev.magicmq.pyspigot.event.custom import CustomEvent
```

### Creating the Custom Event

`CustomEvent` is an object that has three parameters associated with it that can be set when an instance of it is created:

- `name`: The name of the event. Usually, this should be a specific name that an event listener can use to differentiate between multiple `CustomEvent` events coming from different scripts.
- `data`: The data attached to the event. The code example below creates some dummy data for this, but typically this will be some useful data related to the event that an event listener can utilize.
    - *Any* Python type can be used for the `data` parameter; no type conversion is required.
- `async` (optional): If your event is being created and called from an asynchronous context (such as from within an asynchronous task), this should be set to `True`. Otherwise, it can be ommitted or set to `False`.

``` py linenums="1"
from dev.magicmq.pyspigot.event.custom import CustomEvent

dictionary = {'test': '1', 'test2': '2'}

# Create the custom event with the dictionary as the data
event = CustomEvent('script1_event', dictionary)
```

If in an asynchronous context, the event should be created like so:

``` py linenums="1"
from dev.magicmq.pyspigot.event.custom import CustomEvent

dictionary = {'test': '1', 'test2': '2'}

# Create the custom event with the dictionary as the data
event = CustomEvent('script1_event', dictionary, True)
```

### Calling the Custom Event

The event is called with Bukkit's plugin manager, which is accessible via the `Bukkit` class (`org.bukkit.Bukkit`):

``` py linenums="1"
from org.bukkit import Bukkit

...

Bukkit.getPluginManager().callEvent(event)
```

## Code Example

Putting it all together, here is a functional example:

``` py linenums="1"
from dev.magicmq.pyspigot.event.custom import CustomEvent # (1)!
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

## Listening to the Custom Event

Listening to the custom event is done in the same way as you would listen to any other Bukkit/Spigot event. The `CustomEvent` class itself has multiple functions available to use:

- `event.getScript()`: Get the script that created and called this event.
- `event.getName()`: Get the name of the event.
- `event.getData()`: Get the data attached to the event. This is returned as a `PyObject`, which can be used directly from within a script without any type coercion.
- `event.getDataAsType(String clazz)`: Get the data attached to the event as an instance of the provided class name. This method is useful if you can't cast the `PyObject` data to the appropriate Java type.
- `event.getDataAsType(Class<T> clazz)`: Get the data attached to the event as an instance of the provided class. This method is useful if you can't cast the `PyObject` data to the appropriate Java type.

The `CustomEvent` is also [cancellable](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Cancellable.html), which means a listener may cancel the event if desired.

## Summary

- The `CustomEvent` class allows scripts to signal events to other scripts or plugins, with support for passing custom `data` and specifying an event `name`.
- Creating the `CustomEvent` accepts multiple parameters: `name` (event identifier), `data` (any Python type), and `async` (optional, for asynchronous contexts).
- Call the event by using Bukkit's plugin manager via `Bukkit.getPluginManager().callEvent(event)`.
- Other scripts and plugins can listen for `CustomEvent` in the same way as any other Bukkit/Spigot event. Key event functions include `getScript()` (returns the calling script), `getName()` (returns the event name), and `getData()` (returns the data attached to the event).