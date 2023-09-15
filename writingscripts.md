# Scripts

This tutorial provides an overview of PySpigot only, and does not cover in detail the Bukkit/Spigot API or writing Python code. Any questions concerning Python syntax or writing Python code in general should be redirected to the appropriate forum, as this tutorial will not provide an intoroduction to writing basic Python code. 

There are a few basic things to keep in mind when writing PySpigot scripts:

- Under the hood, PySpigot utilizes Jython, a Java implementation of Python. Currently, Jython implements Python 2 only, so Python 2 syntax should be used when writing PySpigot scripts.
- Scripts must be written in Python syntax and script files should in `.py`. Files that do not end in .py will not be loaded.
- Scripts are placed in the `scripts` folder under the PySpigot plugin folder.
- Avoid using the variable names `global` and `logger`. These variable names are assigned automatically at runtime. More information on these below.
- Scripts are functionally isolated from one another. With the exception of the `global` variable (see the [Global Variables](#global-variables) section below), nothing is shared across scripts.
- To make use of any of the managers that PySpigot provides (such as registering listeners, tasks, etc.), they must be imported into your script. See the section below on Making Use of PySpigot's Managersfor details.

## A Note About Jython

Under the hood, PySpigot utilizes Jython, a Java implementation of Python. The PySpigot jar file is quite large in comparison to other Spigot plugins because Jython (as well as its dependencies) are bundled into PySpigot.

Jython is written such that scripts are compiled and interpreted entirely in Java. This means that scripts have native access to the entire Java class path at runtime, making it very easy to work with the Spigot API and other aspects of the server. Consider the following example:

```python
from org.bukkit import Bukkit
from org.bukkit import Location

teleport_location = Location(Bukkit.getWorld('world'), 0, 64, 0)
online_players = Bukkit.getOnlinePlayers() for player in online_players:
player.teleport(teleport_location)
```

As you can see from the above code block, working with Java classes/objects is intuitive. Should you have any trouble interfacing with Java, Jython has fairly well-written documentation you can check out [here](https://jython.readthedocs.io/en/latest/).

Currently, the latest version of Jython implements Python 2. Thus, for now, PySpigot scripts are written in Python 2. While some may see this as a drawback, Python 2 is usually sufficient for the vast majority of use cases of PySpigot, and I have not yet found any case where a Python 3 feature was required for script functionality. The developers of Jython intend on implementing Python 3 in a future release of Jython, but the expected timeframe of this update is unclear. Work is ongoing on the [Jython GitHub repository](https://github.com/jython/jython).

For more information about Jython, visit [jython.org](https://www.jython.org/).

## Standard Python Libraries

Jython *does not* support many of the built-in Python modules (i.e. those that are written in C for Python). These would have to be ported to Java or implemented with a JNI bridge. Some built-in modules have been ported to Jython, most notably `cStringIO`, `cPickle`, `struct`, and `binascii`. Jython's documentation states it is unlikely JNI modules will ever be included in the Jython proper.

Jython now supports a large marjority of the standard Python library. However, Jython's documentation has been slow in keeping up with these additions, so if Jython's documentation does not reference a library, it may still be supported.

If you want to use a standard Python module in your script, try importing it. If that works, then you're probably all set. You can also call `dir()` on the modules to check the list of functions it implements.

## Basic Script Information

All PySpigot scripts are designed to be *self-contained*, single files. This means that each script will, at most, consist of one file only. Additionally, scripts are *isolated* from one another, meaning they do not share variables, functions, or scope. Scripts are capable of interacting with one another in various ways (more detail on this below), but think of each .py file in the `scipts` folder as an individual entity, executed in its own environment.

PySpigot scripts are placed in the `scripts` folder in PySpigot's main plugin folder. PySpigot will attempt to load any file in the `scripts` folder that ends in the `.py` extension. Any files in the `scripts` folder that do not end in `.py` will not be loaded.

## Script Loading

PySpigot loads and runs all scripts in the scripts folder automatically and **in alphabetical order**. This means that if a script depends on another script, then you should name it such that it falls after the script it depends on alphabetically (so it loads after the script it depends on).

There is one config option related to loading scripts:

- `script-load-delay`: This is the delay, in ticks, that PySpigot will wait **after server loading is completed** to load scripts. For example, if the value is 20, then PySpigot will wait 20 ticks (or 1 second) after the server finishes loading to load scripts.

Of course, scripts can also be manually loaded using `/pyspigot load <scriptname>` if you want to load/enable a script after server start/plugin load.

## Start and Stop Functions

There are two special functions you may include in your PySpigot scripts: `start` and `stop`. Both take no parameters.

If a `start` function is defined in your script, it will be called by PySpigot when the script starts.

If a `stop` function is defined in your script, it will be called by PySpigot when your script is stopped/unloaded.

?> Both `start` and `stop` are optional, you do not need to define them in your script if they are not needed.

## PySpigot's Managers

PySpigot provides a variety of managers to more easily work with parts of the Bukkit/Spigot API. For instructions on importing these into your script, see below. PySpigot managers currently include:

- ScriptManager, for loading and unloading scripts from within another script. This is accessed as `script` under PySpigot, or imported individually as `dev.magicmq.pyspigot.manager.script.ScriptManager`.
- ListenerManager, for registering event listeners. This is accessed as `listener` under PySpigot, or imported individually as `dev.magicmq.pyspigot.manager.listener.ListenerManager`.
- CommandManager, for registering and working with commands. This is accessed as `commmand` under PySpigot, or imported individually as `dev.magicmq.pyspigot.manager.command.CommandManager`
- TaskManager, for registering a variety of repeating, delayed, and asynchronous tasks. This is accessed as `scheduler` under PySpigot, or imported individually as `dev.magicmq.pyspigot.manager.task.TaskManager`
- ConfigManager, for working with configuration files. This is accessed as `config` under PySpigot, or imported individually as `dev.magicmq.pyspigot.manager.config.ConfigManager`
- ProtocolManager, to work with ProtocolLib. This is accessed as `protocol` under PySpigot, or imported individually as `dev.magicmq.pyspigot.manager.protocol.ProtocolManager`
- PlaceholderManager, to work with PlaceholderAPI. This is accessed as `placeholder` under PySpigot, or imported individually as `dev.magicmq.pyspigot.manager.placeholder.PlaceholderManager`

The following table summarizes how to access managers:

| Manager             | Access Under PySpigot       | Standalone Import                                                         |
| ------------------- | --------------------------- | ------------------------------------------------------------------------- |
| Script Manager      | `PySpigot.script`           | `from dev.magicmq.pyspigot.manager.script import ScriptManager`           |
| Listener Manager    | `PySpigot.listener`         | `from dev.magicmq.pyspigot.manager.listener import ListenerManager`       |
| Command Manager     | `PySpigot.command`          | `from dev.magicmq.pyspigot.manager.command import CommandManager`         |
| Task Manager        | `PySpigot.scheduler`        | `from dev.magicmq.pyspigot.manager.task import TaskManager`               |
| Config Manager      | `PySpigot.config`           | `from dev.magicmq.pyspigot.manager.config import ConfigManager`           |
| Protocol Manager    | `PySpigot.protocol`         | `from dev.magicmq.pyspigot.manager.protocol import ProtocolManager`       |
| Placeholder Manager | `PySpigot.placeholder`      | `from dev.magicmq.pyspigot.manager.placeholder import PlaceholderManager` |

!> The Protocol Manager and Placeholder Manager are *optional* managers. These managers should only be accessed if the ProtocolLib and/or PlaceholderAPI plugins are loaded and enabled.

To utilize these managers, they must be imported into your script. This can be done in two ways:

### Import all managers at once using the PySpigot class

This is the preferred way to import managers as less code is required:

```python
from dev.magicmq.pyspigot import PySpigot as ps

ps.script.<function>
ps.listener.<function>
ps.command.<function>
ps.scheduler.<function>
ps.config.<function>
ps.protocol.<function>
ps.placeholder.<function>
```

In the above code, PySpigot is imported as ps. Managers are called using their simplified name, `script` for ScriptManager, `listener` for ListenerManager, `command` for CommandManager, `scheduler` for TaskManager, `config` for ConfigManager, and `protocol` for ProtocolManager.

### Import each manager individually:

```python
from dev.magicmq.pyspigot.manager.script import ScriptManager as script
from dev.magicmq.pyspigot.manager.listener import ListenerManager as listener
from dev.magicmq.pyspigot.manager.command import CommandManager as command
from dev.magicmq.pyspigot.manager.task import TaskManager as scheduler
from dev.magicmq.pyspigot.manager.config import ConfigManager as config
from dev.magicmq.pyspigot.manager.protocol import ProtocolManager as protocol

script.get().<function>
listener.get().<function>
command.get().<function>
...
```

!> If importing a manager individually, `get()` *must* be used each time the manager is called!

## Global Variables

PySpigot assigns a variable to the local namespace called `global` that is available to all loaded scripts. On the Java end, this variable is a `HashMap`, which stores data in key:value pairs, much like a dict in Python. The intention of this variable is to act as a global set of variables. This is a nifty feature if you would like to share variables/values across multiple different scripts.

Changes to variables inserted into this global set are automatically visible to all scripts. There is no need to re-insert a variable into the global set of variables if its value changes.

-   `global.put(name, value)`: Inserts a new value into the global set of variables with the given name.
-   `global.get(name)`: Retrieves a value from the global set of variables. Will return `None` if no value is found.
-   `global.remove(name)`: Removes a value from the global set of variables with the given name.
-   `global.containsKey(name)`: Returns `True` if there is a value in the set of global variables with the given name, `False` if otherwise.

!> Names are unique. If a new value is inserted into the set of global values with the same name as an existing value, then the old value will be overridden and inevitably lost.

For more advanced usage, see the [JavaDocs for HashMap](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html) for a complete list of available functions.

## Script Errors

Scripts can generate errors/exceptions. PySpigot will attempt to handle these to prevent other parts of your script from breaking. If a script happens to generate an unhandled error/exception when it is loaded, the script will be automatically unloaded to prevent further issues. If an unhandled error/exception occurs somewhere else at a later point in time, such as while calling an event listener or command function, the script will remain loaded, but subsequent code within the function will not be executed. In any case, errors/exceptions will be logged to the console and to the respective script's log file (if file logging is enabled in the `config.yml`). You may use Python's `try:` and `except:` syntax to handle exceptions yourself. This will work for Java exceptions as well.

There are two types of errors that a script can produce:

### Python Exceptions

These are exceptions intrinsic to your script's Python code. These exceptions will generate a log entry with a Python traceback indicating the script file and line that caused the exception. Because these exceptions originate in Python code, they should be fairly easy to debug. They will look like this:

![An example of what a Python exception looks like in the server console.](images/python_exception.png)

The boxed text is the Python traceback.

### Java Exceptions

These exceptions occur when a script calls Java code and the exception occurs somewhere within the Java code (but not from within the script). These exceptions will also generate a log entry with a Python traceback indicating the script file and line that caused the exception. These can be trickier to debug because the cause of the exception may not be immediately apparent. The script log/console should give you an idea of what went wrong. They will look like this:

![An example of what a Java exception looks like in the server console.](images/java_exception.png)

You'll notice that these look very similar to Python exceptions. The only difference is that there will be an accompanying Java exception (`java.lang.<exception>`) along with a brief message about why the exception occurred.

### Final Note About Exceptions

Because PySpigot is an active project in youth stages of development, you may encounter exceptions that are caused by a bug within PySpigot itself. If something goes wrong with your script, and your debuging efforts have been futile, please [submit an issue on Github](https://github.com/magicmq/PySpigot/issues).

## Script Logging

Like scripts themselves, a script's logger is self-contained. Each script has its own logger, which is a subclass of [java.util.logging.Logger](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Logger.html).

If you would like to print log messages in your script, it is highly recommended to use a script's respective logger. Using Python's `print` function works, but it will not automatically indicate which script the message came from (unless if you add this to the message yourself). `print` will also not log messages to a script's log file.

To access your script's logger, use the `logger` variable. PySpigot automatically assigns this when a script is loaded for convenience.

When accessing your script's logger, you can use any of the functions listed [here](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Logger.html). PySpigot adds two additional functions for convenience:

- `logger.print(message)`: Useful for quickly adding debug messages to your script
- `logger.debug(message)`: Under the hood, functions the same as `logger.print`

### Script Log Files

As stated above, each script has its own log file, and these can be found in the `logs` folder within the PySpigot plugin folder. All messages related to a script will be logged here. If you would like to disable file logging for scripts, set the `log-to-file` value to `false` in PySpigot config.yml.

You may also change which messages are logged to a script's log file. To do so, edit the `min-log-level` value in the config.yml. Use Java's [Logging levels](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Level.html).

You may also change the format of time stamps within script logs files. To do so, edit the `log-timestamp-format` value in the config.yml. Use a format that conforms to Java's [DateTimeFormatter](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html).