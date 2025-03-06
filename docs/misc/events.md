# PySpigot Events

PySpigot currently has four events that you may listen for in your plugins.

???+ note

    It is *possible* that script events may fire from an asynchronous context (for example, if a script exception occurs within an asynchronous task). To check if the event is within an asynchronous context, use `script.isAsynchronous()`. This method returns `true` if the event context is asynchronous, `false` if it is not.

## `ScriptEvent`

This is the base script event for PySpigot. This event will fire when a script is loaded, unloaded, or an exception occurs within a script. `ScriptExceptionEvent`, `ScriptLoadEvent`, and `ScriptUnloadEvent` outlined below are all subclasses of this event.

Call `event.getScript()` to get the script associated with the event.

## `ScriptExceptionEvent`

This event is called when an exception (either a Java or a Python exception) occurs within a script.

This event has four methods available to use:

- `event.getScript()`: Get the script that threw the exception.
- `event.getException()`: Returns a [`PyException`](https://javadoc.io/doc/org.python/jython/latest/org/python/core/PyException.html) object representing the exception that was thrown.
- `event.doReportException()`: Returns `true` if the exception should be logged to console and the script's log file, false if otherwise
- `event.setReportException(boolean reportException)`: Set if the exception should be logged to console and the script's log file.

## `ScriptLoadEvent`

This event is called when a script is loaded. The event will fire only if the script loads successfully, I.E. the `RunResult` is `RunResult.SUCCESS`.

Call `event.getScript()` to get the script associated with the event.

## `ScriptUnloadEvent`

This event is called when a script is unloaded.

Call `event.getScript()` to get the script associated with the event.

## Custom Script Event

PySpigot also includes a custom event (`CustomEvent`) that scripts may instantiate and call. This event is designed for easier interplay between scripts and plugins.

This event can be listened for just as any other Bukkit/Spigot event normally would. It contains several methods available to use:

- `event.getScript()`: Get the script that created and called this event.
- `event.getName()`: Get the "name" of this event. This variable can be used for increased granularity of the custom event. Since multiple scripts can use this event for different purposes, the event name can be used to differentiate between different use cases.
- `event.getData()`: Get the data attached to this event. This is returned as a `PyObject`, which, in most cases, can be simply casted to a Java type.
- `event.getDataAsType(String clazz)`: Get the data attached to the event as an instance of the provided class name. This method is useful if you can't cast the `PyObject` data to the appropriate Java type.
- `event.getDataAsType(Class<T> clazz)`: Get the data attached to the event as an instance of the provided class. This method is useful if you can't cast the `PyObject` data to the appropriate Java type.

`CustomEvent` also implements [`Cancellable`](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Cancellable.html), and thus `event.isCancelled()` and `event.setCancelled(boolean cancelled)` are available to use as well.

???+ warning

    The `CustomEvent` should be instantiated from scripts only. Attempts to instantiate the event outside of a script context will result in an exception.

### Type Coercion

Jython normally handles type coercion automatically when going from a Python context to a Java context. As stated previously, most Python types can be simply casted to Java types. If you can't cast, then try using one of the `event.getDataAsType` methods from `CustomEvent` to manually convert the `PyObject` to the desired type. If this doesn't work, then the types likely aren't interchangeable, and you'll need to use a different type for the data. For a complete list of Python types and their Java counterparts, see the [Type Coercion](types.md) page.

### Code Example

Conside the following PySpigot script:

``` py linenums="1"
from dev.magicmq.pyspigot.event.custom import CustomEvent
from org.bukkit import Bukkit

dictionary = {'test': '1', 'test2': '2'} # (1)!

event = CustomEvent('test_event', dictionary) # (2)!

Bukkit.getPluginManager().callEvent(event) # (3)!
```

1. Here, we create a new `dict` and populate it with some data.

2. Here, we create a new instance of `CustomEvent`, assigning "test_event" for the name, and the dictionary we created previously as the data.

3. Here, we call the event by using Bukkit's plugin manager.

Then, in a Java plugin, we could create an event listener for this event that handles the event and does something with the data attached to it:

``` java linenums="1"
import org.bukkit.event.Listener;
import org.bukkit.event.EventHandler;
import dev.magicmq.pyspigot.event.custom.CustomEvent

public class PluginListener implements Listener {

    @EventHandler
    public void onCustomEvent(CustomEvent event) {
        String name = event.getName();
        if (name.equals("test_event")) { // (1)!
            PyObject data = event.getData()
            Map data_map = (Map) data; // (2)!

            for (Object key : data_map.keySet()) { // (3)!
                System.out.println(key + ": " + data_map.get(key));
            }
        }
    }
}
```

1. Here, we check if the event name is equal to "test_event", which is the name we assigned to the event when we called it from our script. Again, using the event name is useful if there are multiple scripts utilizing the `CustomEvent`, as scripts could set different names depending on context, and plugins could check the name as is being done here.

2. Here, we cast the data, a `PyObject`, to a map. Since we know the data coming from this event is a python `dict`, then we can safely cast, as the Python `dict` type is equivalent to a Java `Map` in Jython.

3. Here, we loop through each element in the map, and print the key as well as its associated value.

## Conclusion

See the [PySpigot JavaDocs](https://javadocs.magicmq.dev/pyspigot/dev/magicmq/pyspigot/event/package-summary.html) for complete API documentation on script events.