# Scripts

This tutorial provides an overview of PySpigot only, and does not cover in detail the Bukkit/Spigot API or writing Python code. Any questions concerning Python syntax or writing Python code in general should be redirected to the appropriate forum, as this tutorial will not provide an intoroduction to writing basic Python code. 

There are a few basic things to keep in mind when writing PySpigot scripts:

- PySpigot officially supports Spigot and Paper on Minecraft versions 1.12.2 and newer.
- Under the hood, PySpigot utilizes Jython, a Java implementation of Python. Currently, Jython implements Python 2 only, so Python 2 syntax should be used when writing PySpigot scripts.
- Scripts must be written in Python syntax and script files should in `.py`. Files that do not end in .py will not be loaded.
- Scripts are placed in the `scripts` folder under the PySpigot plugin folder. PySpigot allows for creation of subfolders within the scripts folder for organizational purposes, but script names must be unique across all subfolders.
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

PySpigot scripts are placed in the `scripts` folder, which can be found in PySpigot's main plugin folder. Creation of subfolders within the `scripts` folder for organizational purposes is supported. PySpigot will attempt to load any file in the `scripts` folder (including in subfolders) that ends in the `.py` extension. Any files in the `scripts` folder that do not end in `.py` will not be loaded.

!> Script names must be unique, as their names are used to identify them at runtime. This caveat also applies if you are using subfolders within the `scripts` folder. For example, `scripts/folder1/test.py` and `scripts/folder2/test.py` will conflict, but `scripts/folder1/test.py` and `scripts/folder2/test2.py` will not.

## Script Options

There are a variety of options that can be set for each script, including whether or not it is enabled, dependencies, and logging options. These are set within the `script_options.yml` file in PySpigot's main plugin folder. For more information on script options, see the [Script Options](scriptoptions.md) page.

!> Defining script options for each script is *optional*; scripts will function normally without explicitly-defined options.

## Script Loading

PySpigot loads and runs all scripts in the scripts folder (including scripts within subfolders) automatically on plugin load or server start. Script load order is determined by script dependencies as defined in the `script_options.yml` file. Scripts that don't list any dependencies are loaded in no predetermined order (randomly).

Scripts can also be manually loaded using `/pyspigot load <scriptname>` if you want to load/enable a script after server start/plugin load. If you make changes to a script during runtime, you must reload it for changes to take effect. Reload scripts with `/pyspigot reload <scriptname>`.

There is one config option related to loading scripts:

- `script-load-delay`: This is the delay, in ticks, that PySpigot will wait **after server loading is completed** to load scripts. There are 20 server ticks in one real-world second. For example, if the value is 20, then PySpigot will wait 20 ticks (or 1 second) after the server finishes loading to load scripts.

More documentation on this module will be added later.

## Script Permissions

PySpigot allows scripts to define a list of permissions that it uses. This is useful if scripts want to restrict access to certain features. Script permissions are initialized and loaded just prior to parsing and executing the script's code, and are removed just after a script is stopped.

Script permissions are defined in the `script_options.yml` file. For more information on how to define permissions, see the [documentation for script options](scriptoptions.md#permissions).

## Start and Stop Functions

There are two special functions you may include in your PySpigot scripts: `start` and `stop`. Both take no parameters.

If a `start` function is defined in your script, it will be called by PySpigot when the script starts.

If a `stop` function is defined in your script, it will be called by PySpigot when your script is stopped/unloaded.

?> Both `start` and `stop` are optional, you do not need to define them in your script if they are not needed.

## The pyspigot Helper Module

As of version 0.5.0, PySpigot ships with a helper module called `pyspigot.py` that contains various useful functions to access PySpigot's manager classes. This module is automatically placed into the `python-libs` folder on plugin load.

PySpigot includes an automated system to automatically update the `pyspigot.py` helper module when changes are detected. This feature can be disabled if desired by setting the `auto-pyspigot-lib-update-enabled` option under `debug-options` in the config.yml file to `false`. It is recommended that you leave this enabled to ensure that any changes, additions, and fixes are always reflected on the user end.

[Click here](https://github.com/magicmq/pyspigot/blob/master/src/main/resources/python-libs/pyspigot.py) to view the source code of the `pyspigot.py` module.

## PySpigot's Managers

PySpigot provides a variety of managers to more easily work with parts of the Bukkit/Spigot API. For instructions on importing these into your script, see below. PySpigot managers currently include:

- ScriptManager, for loading and unloading scripts from within another script.
- ListenerManager, for registering event listeners.
- CommandManager, for registering and working with commands.
- TaskManager, for registering a variety of repeating, delayed, and asynchronous tasks.
- ConfigManager, for working with configuration files.
- ProtocolManager, to work with ProtocolLib.
- PlaceholderManager, to work with PlaceholderAPI.
- DatabaseManager, to connect to and interact with SQL-type and Mongo databases.
- RedisManager, to connect to and interact with a Redis server instance.

Managers must be imported into your script inn order for you to access them. The following table summarizes how to access managers, but read the sections below for more detail on how to import them:

| Manager             | Access Via Helper Module         | Access Under PySpigot       | Standalone Import                                                         |
| ------------------- | -------------------------------- | --------------------------- | ------------------------------------------------------------------------- |
| Script Manager      | `pyspigot.script_manager()`      | `PySpigot.script`           | `from dev.magicmq.pyspigot.manager.script import ScriptManager`           |
| Listener Manager    | `pyspigot.listener_manager()`    | `PySpigot.listener`         | `from dev.magicmq.pyspigot.manager.listener import ListenerManager`       |
| Command Manager     | `pyspigot.command_manager()`     | `PySpigot.command`          | `from dev.magicmq.pyspigot.manager.command import CommandManager`         |
| Task Manager        | `pyspigot.task_manager()`        | `PySpigot.scheduler`        | `from dev.magicmq.pyspigot.manager.task import TaskManager`               |
| Config Manager      | `pyspigot.config_manager()`      | `PySpigot.config`           | `from dev.magicmq.pyspigot.manager.config import ConfigManager`           |
| Protocol Manager    | `pyspigot.protocol_manager()`    | `PySpigot.protocol`         | `from dev.magicmq.pyspigot.manager.protocol import ProtocolManager`       |
| Placeholder Manager | `pyspigot.placeholder_manager()` | `PySpigot.placeholder`      | `from dev.magicmq.pyspigot.manager.placeholder import PlaceholderManager` |
| Database Manager    | `pyspigot.database_manager()`    | `PySpigot.database`         | `from dev.nagicmq.pyspigot.manager.database import DatabaseManager`       |
| Redis Manager       | `pyspigot.redis_manager()`       | `PySpigot.redis`            | `from dev.magicmq.pyspigot.manager.redis import RedisManager`             |

!> The Protocol Manager and Placeholder Manager are *optional* managers. These managers should only be accessed if the ProtocolLib and/or PlaceholderAPI plugins are loaded and enabled.

To utilize these managers, they must be imported into your script. This can be done in three ways:

### Import managers via the pyspigot.py helper library

As of version 0.5.0, PySpigot ships with a `pyspigot.py` helper module, which is automatically placed into the `python-libs` folder when the plugin is initialized. This module contains several functions to allow for access to all managers.

```python
import pyspigot as ps

ps.script_manager().<function>
ps.listener_manager().<function>
ps.command_manager().<function>
ps.task_manager().<function>
ps.config_manager().<function>
ps.protocol_manager().<function>
ps.placeholder_manager().<function>
ps.database_manager().<function>
ps.redis_manager().<function>
```

In the above code, the PySpigot library is imported as `ps`. Then, functions within the library are called to get each manager. Of course, you can also assign the needed managers to a variable for ease of use in multiple locations within your code, like so:

```python
import pyspigot as ps

script = ps.script_manager()
listener = ps.listener_manager()
command = ps.command_manager()
...

script.<function>
listener.<function>
command.<function>
...
```

The PySpigot helper module also includes a series of convenience variables for ease of access of managers. You'll see these common aliases used in example code throughout the documentation for PySpigot.

```python
import pyspigot as ps
```

- ScriptManager: `ps.script`, `ps.scripts`, `ps.sm`
- ListenerManager: `ps.listener`, `ps.listeners`, `ps.lm`, `ps.event`, `ps.events`, `ps.em`
- CommandManager: `ps.command`, `ps.commands`, `ps.cm`
- TaskManager: `ps.scheduler`, `ps.scm`, `ps.tasks`, `ps.tm`
- ConfigManager: `ps.config`, `ps.configs`, `ps.com`
- ProtocolManager: `ps.protocol`, `ps.protocol_lib`, `ps.protocols`, `ps.pm`
- PlaceholderManager: `ps.placeholder`, `ps.placeholder_api`, `ps.placeholders`, `ps.plm`
- DatabaseManager: `ps.database`
- RedisManager: `ps.redis`

### Import all managers at once using the PySpigot class

This used to be the preferred way to access managers, but is no longer the preferred method as of version 0.5.0. This method is nevertheless still functional and can be used if desired.

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

You can also import each manager class individually, directly from PySpigot's Java code.

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

PySpigot assigns a variable to the local namespace called `global` that is available to all loaded scripts. On the Java end, this variable is a `HashMap`, which stores data in key:value pairs, much like a dict in Python. The intention of this system is to act as a global set of variables. This is a nifty feature if you would like to share variables/values across multiple different scripts.

See the [Global Variables](globalvariables.md) page for detailed information on how to use the global variables system.

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

PySpigot creates a new logger for each running script. A script's logger is automatically assigned to its global namespace under the variable name `logger`. To access your script's logger, use the `logger` variable.

If you would like to print log messages in your script, it is highly recommended to use a script's respective logger. Using Python's `print` function works, but it will not automatically indicate which script the message came from (unless if you add this to the message yourself). `print` will also not log messages to a script's log file.

When accessing your script's logger, you can use any of the functions listed [here](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Logger.html). PySpigot adds two additional functions for convenience:

- `logger.print(message)`: Useful for quickly adding debug messages to your script
- `logger.debug(message)`: Under the hood, functions the same as `logger.print`

### Script Log Files

As stated above, each script has its own log file, and these can be found in the `logs` folder within the PySpigot plugin folder. All messages related to a script will be logged here. If you would like to disable file logging for scripts, set the `log-to-file` value to `false` in PySpigot config.yml.

You may also change which messages are logged to a script's log file. To do so, edit the `min-log-level` value in the config.yml. Use Java's [Logging levels](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Level.html).

You may also change the format of time stamps within script logs files. To do so, edit the `log-timestamp-format` value in the config.yml. Use a format that conforms to Java's [DateTimeFormatter](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html).

## Non-ASCII Characters in Script Files

Jython reads and compiles script files using ASCII encoding. This means that it won't recognize non-ASCII characters in the file. You may see a `SyntaxError` when you load a script with non-ASCII characters. Additionally, in Python 2, the `str` type is a collection of 8-bit characters. Consequently, all characters in the English alphabet (and some basic symbols) can be represented using these 8-bit characters, but special symbols and characters from non-Latin alphabets cannot. There are a couple ways to work around these two constraints:

### Workaround 1

Jython allows you to specify the encoding of your script file. This is done by specifying an [encoding declaration](https://docs.python.org/2/reference/lexical_analysis.html#encoding-declarations) in your script. This ensures that when Jython reads and compiles the file, it will recognize the non-ASCII characters. Add the following to the *first or second line* of your script file:

`#coding: utf-8`

Of course, you can replace `utf-8` with whatever character encoding standard you'd like Jython to use. For a list of supported encoding schemes, see [this page](https://docs.python.org/2/library/codecs.html#standard-encodings).

!> You must put the encoding declaration on either the first or second line of your script, as Jython only searches for encoding declarations in this area.

Python 2 includes a `unicode` type, which supports all UTF-8 characters (symbols, non-Latin alphabets, etc.). You can specify that you want to use the `unicode` type for the string (and not `str`) by adding a preceding `u`. For example:

```python
#coding: utf-8

from org.bukkit import Bukkit

Bukkit.broadcastMessage(u'Привет, мир!')
```

Notice that a `u` directly proceeds the string. This denotes that the string should be treated as a unicode string.

### Workaround 2

You can use also use the hex codes of the unicode characters you want to write. You will still need to denote that the string is a unicode string by adding a preceding `u`, but you don't need to add an encoding declaration at the top of your script file since you aren't actually writing any non-ASCII characters in the file. For example:

```python
from org.bukkit import Bukkit

Bukkit.broadcastMessage(u'\u041f\u0440\u0438\u0432\u0435\u0442, \u043c\u0438\u0440!')
```