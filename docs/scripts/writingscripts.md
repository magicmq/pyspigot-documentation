# Writing Scripts

This tutorial provides an overview of PySpigot only, and does not cover in detail the underlying API or writing Python code in general. Any questions concerning Python syntax or writing Python code in general should be redirected to the appropriate forum, as this tutorial will not provide an intoroduction to writing basic Python code. 

There are a few basic things to keep in mind when writing PySpigot scripts:

- As of version 0.9.1, PySpigot requires Java 21 or above.
- PySpigot officially supports Spigot and Paper on Minecraft version 1.16 and newer, Velocity version 3.0.0 and newer, and BungeeCord versions corresponding to Minecraft versions 1.16 and newer.
- Under the hood, PySpigot utilizes Jython, a Java implementation of Python. Currently, Jython implements Python 2 only, so Python 2 syntax should be used when writing PySpigot scripts.
- Scripts must be written in Python syntax and script files should in `.py`. Files that do not end in .py will not be loaded.
- Scripts are placed in the `scripts` folder under the PySpigot plugin folder. PySpigot allows for creation of subfolders within the scripts folder for organizational purposes, but script names must be unique across all subfolders and projects.
- Projects are placed in the `projects` folder under the PySpigot plugin folder. 
- Avoid using the variable names `global` and `logger`. These variable names are assigned automatically at runtime. More information on these below.
- Scripts and projects are functionally isolated from one another. With the exception of the `global` variable (see the [Global Variables](#global-variables) section below), nothing is shared across scripts or projects.
- To make use of any of the managers that PySpigot provides (such as registering listeners, tasks, etc.), they must be imported into your script. See the [managers](../managers/usage.md) page for more details.
- If you are utilizing the API of any plugin other than ProtocolLib or PlaceholderAPI, make sure you specify the plugin as a dependency in the `script_options.yml` file. See the [Script Options](scriptoptions.md#plugin-depend) page for more info.

???+ notice

    This page covers core fundamentals for writing both single-file scripts and multi-file projects. However, writing multi-file projects carries some key differences to be aware of. For more information, see the [writing projects](../projects/writingprojects.md) page.

## A Note About Jython

Under the hood, PySpigot utilizes Jython, a Java implementation of Python. The PySpigot jar file is quite large in comparison to other Bukkit/Velocity/bungeeCord plugins because Jython (as well as its dependencies) are bundled into the PySpigot JAR file.

Jython is written such that scripts are compiled and interpreted entirely in Java. This means that scripts have native access to the entire Java class path at runtime, making it very easy to work with the underlying platform API and other aspects of the server. Consider the following example (on the Bukkit platform):

``` py linenums="1"
from org.bukkit import Bukkit
from org.bukkit import Location

teleport_location = Location(Bukkit.getWorld('world'), 0, 64, 0)

online_players = Bukkit.getOnlinePlayers()

for player in online_players:
    player.teleport(teleport_location)
```

As you can see from the above code block, working with Java classes/objects is intuitive. Should you have any trouble interfacing with Java, Jython has fairly well-written documentation you can check out [here](https://jython.readthedocs.io/en/latest/).

By default, Jython internals are initialized on server startup (when PySpigot is loaded). This can result in a momentary hang during startup when PySpigot is loading. This also results in some memory overhead, even if no scripts are loaded. To disable this feature, and instead delay initialization of Jython until script loading, set `init-on-startup` to `false` in the `jython-options` section of the [PySpigot config file](../pyspigot/pluginconfiguration.md#init-on-startup).

Currently, the latest version of Jython implements Python 2. Thus, for now, PySpigot scripts are written in Python 2. While some may see this as a drawback, Python 2 is usually sufficient for the vast majority of use cases of PySpigot, and I have not yet found any case where a Python 3 feature was required for script functionality. The developers of Jython intend on implementing Python 3 in a future release of Jython, but the expected timeframe of this update is unclear. Work is ongoing on the [Jython GitHub repository](https://github.com/jython/jython).

For more information about Jython, visit [jython.org](https://www.jython.org/).

## Standard Python Libraries

Jython *does not* support many of the built-in Python modules (i.e. those that are written in C for Python). These would have to be ported to Java or implemented with a JNI bridge. Some built-in modules have been ported to Jython, most notably `cStringIO`, `cPickle`, `struct`, and `binascii`. Jython's documentation states it is unlikely JNI modules will ever be included in the Jython proper.

Jython now supports a large marjority of the standard Python library. However, Jython's documentation has been slow in keeping up with these additions, so if Jython's documentation does not reference a library, it may still be supported.

If you want to use a standard Python module in your script, try importing it. If that works, then you're probably all set. You can also call `dir()` on the modules to check the list of functions it implements.

## Basic Script Information

All PySpigot scripts are designed to be *self-contained*, single files. This means that each script will, at most, consist of one file only. Additionally, scripts are *isolated* from one another, meaning they do not share variables, functions, or scope. Scripts are capable of interacting with one another in various ways (more detail on this below), but think of each .py file in the `scipts` folder as an individual entity, executed in its own environment.

PySpigot scripts are placed in the `scripts` folder, which can be found in PySpigot's main plugin folder. Creation of subfolders within the `scripts` folder for organizational purposes is supported. PySpigot will attempt to load any file in the `scripts` folder (including in subfolders) that ends in the `.py` extension. Any files in the `scripts` folder that do not end in `.py` will not be loaded.

???+ warning

    Script names must be unique across other script names *and* project names, as their names are used to identify them at runtime. This caveat also applies if you are using subfolders within the `scripts` folder. For example, `scripts/folder1/test.py` and `scripts/folder2/test.py` will conflict, but `scripts/folder1/test.py` and `scripts/folder2/test2.py` will not.

## Script Options

There are a variety of options that can be set for each script, including whether or not it is enabled, load priority, and logging options. These are set within the `script_options.yml` file in PySpigot's main plugin folder. For more information on script options, see the [Script Options](scriptoptions.md) page.

???+ notice

    Defining script options for each script is *optional*; scripts will function normally without explicitly-defined options.

### Script Permissions

???+ warning

    Script permissions are not supported on Velocity and BungeeCord, so configuring script permissions on these platforms has **no effect**.

**On the Bukkit platform only**, PySpigot allows scripts to define a list of permissions that it uses. This is useful if scripts want to restrict access to certain features. Script permissions are initialized and loaded just prior to parsing and executing the script's code, and are removed just after a script is stopped.

Script permissions are defined in the `script_options.yml` file. For more information on how to define permissions, see the [documentation for script options](scriptoptions.md#permissions).

## Script Loading

PySpigot loads and runs all scripts in the scripts folder (including scripts within subfolders) automatically on plugin load or server start. Script load order is determined by load priority, as defined in the `script_options.yml` file. Scripts that don't specify a load priority will inherit the default load priority specified in the `script-option-defaults` section of the `config.yml`. Scripts that have the same load priority are loaded in alphabetical order.

Scripts can also be manually loaded using the load command:

- On Bukkit: `/pyspigot load <scriptname>`
- On Velocity: `/pyvelocity load <scriptname>`
- On BungeeCord: `/pybungee load <scriptname>`

 If you make changes to a script during runtime, the script must be reloaded for changes to take effect. Reload scripts with the reload command:

- On Bukkit: `/pyspigot reload <scriptname>`
- On Velocity: `/pyvelocity reload <scriptname>`
- On BungeeCord: `/pybungee reload <scriptname>`

There is one config option related to loading scripts:

- `script-load-delay`: This is the delay, in ticks, that PySpigot will wait **after server loading is completed** to load scripts and projects. There are 20 server ticks in one real-world second. For example, if the value is 20, then PySpigot will wait 20 ticks (or 1 second) after the server finishes loading to load scripts and projects. To disable the load delay, set this value to `0` or `-1`.

???+ notice

    Scripts and projects are interlaced when loading. In other words, they are loaded together. This means that the load priorities of scripts and projects are compared simultaneously. A project with a higher load priority would load earlier than a script with a lower load priority, and vice versa.

## Script Unloading

Scripts can be manually unloaded using the unload command:

- On Bukkit: `/pyspigot unload <scriptname>`
- On Velocity: `/pyvelocity unload <scriptname>`
- On BungeeCord: `/pybungee unload <scriptname>`

Additionally, running the reload command (`/pyspigot reload <scriptname>` on Bukkit) will unload a script first before loading it again (if it was running beforehand).

### Unloading A Script from Within Itself

Unloading a script from within itself can be accomplished in the same way a running script would be terminated in a regular Python environment, via usage of the `sys.exit` function:

``` py
import sys

sys.exit(0)
```

Internally, calling `sys.exit` raises a `SystemExit` exception. PySpigot catches this exception and performs its standard unloading tasks to unload the script that raised the exception.

If you want to unload your script with a signal that an error occured, pass `1` to `sys.exit`. Doing so will prevent the script's `stop` function from being called on unload.

???+ warning

    *Do not* use the script manager to unload a script from within itself! This will lead to unexpected bugs/issues.

## Start and Stop Functions

There are two special functions you may include in your PySpigot scripts: `start` and `stop`.

The `start` function is called automatically by PySpigot when your script loads. Likewise, the `stop` function is called automatically by PySpigot when your script unloads. If your script is unloaded as a result of an error, the `stop` function is *not* called. This error condition also includes unloading a script via `sys.exit` with an exit code of `1`.

The `start` and `stop` functions can accept either zero or one parameter:

- If you define one parameter, PySpigot will pass the [Script Object](../managers/scripts.md#the-script-object) to the function. This object is the representation of the loaded script at runtime. This allows you to obtain information about the script, as well as other key functions, including logging, the script file, and more within the `start` and/or `stop` function.
- If you define zero parameters, PySpigot will not pass any arguments to the function.

???+ notice

    The `start` and `stop` functions are optional. You do not need to define them in your script if they are not needed.

## The `pyspigot.py` Helper Module

PySpigot ships with a helper module called `pyspigot.py` that contains various useful functions to access PySpigot's manager classes. This module is accessible via a simple import:

``` py
import pyspigot as ps

...
```

For more information, see the [PySpigot Helper Module](helpermodule.md) page.

## Global Variables

PySpigot includes a global variable system that allows variables to be shared across different scripts. On the Java end, this system is backed by a `HashMap`, which stores data in key:value pairs, much like a dict in Python. The intention of this system is to act as a global set of variables in the event that multiple scripts need to access the same variable. This is a nifty feature if you would like to share variables/values across multiple different scripts.

See the [Global Variables](globalvariables.md) page for detailed information on how to use the global variables system.

## Script Errors and Exceptions

Scripts can generate errors/exceptions. PySpigot handles all exceptions generated by scripts, in order to: 1) log the exception to console and the script's logger, and 2) to terminate the script's execution if the exception was fatal.

If a script happens to generate an unhandled error/exception when it is loaded, the script will be automatically unloaded to prevent further issues. If an unhandled error/exception occurs somewhere else at a later point in time, such as during execution of an event listener or command executor, the script will remain loaded, but subsequent code within the function will not be executed (unless if, of course, the exception is surrounded with a `try`/`except` block).

In any case, errors/exceptions will be logged to the console and to the respective script's log file (if the script has file logging enabled per its script options).

???+ notice

    Because PySpigot is an active project in early stages of development, you may encounter exceptions that are caused by a bug within PySpigot itself. If something goes wrong with your script, and your debuging efforts have been futile, please [submit an issue on Github](https://github.com/magicmq/PySpigot/issues) or [join the Discord](https://discord.gg/f2u7nzRwuk) to ask for help.

There are two types of errors that a script can produce:

### Python Errors

These exceptions occur when there is an error/exception raised from **within the script's own code**. Because these exceptions originate in Python code, debugging them is fairly straightforward. A traceback will be shown along with the error that was raised. Here is an example of what a Python error would look like:

![An example of what a Python exception looks like in the server console.](../assets/images/python_exception.png)

### Java Exceptions

These exceptions occur when a script calls Java code and the exception occurs somewhere in the **Java code that was called** (but not from within the script itself). These exceptions will also generate a log entry with a Python traceback indicating the script file and line that caused the exception. Here is an example of what a Java exception would look like:

![An example of what a Java exception looks like in the server console.](../assets/images/java_exception.png)

These look similar to Python exceptions, however, in addition to a Python traceback, there will be an accompanying Java exception and stack trace. The message that accompanies the Java exception (boxed in red in the above image) typically provides additional details about why the exception occurred. For example, the message accompanying the above exception states that the command "test" has already been registered (I.E. the script attempted to register the same command twice).

PySpigot routinely throws these types of exceptions (in the form of a `ScriptRuntimeException`) for erroneous/non-permitted operations that scripts attempt to perform, such as registering two different listeners for the same event, or registering two commands with the same name. A `ScriptRuntimeException` could also be thrown if some unhandled exception occurs in PySpigot's internals, such as an exception that occurred when registering an event listener with the platform's API.

???+ note

    Java exceptions may also have a *cause*, which essentially means the execption was thrown because of another exception (thrown earlier). If there is a cause, the cause would also be included in the error message (along with a message and stack trace).

### Handling Exceptions

You may use Python's `try` and `except` syntax in order to catch both Python and Java exceptions yourself. For example, to catch a Java exception:

``` py linenums="1"
import pyspigot as ps
from dev.magicmq.pyspigot.exception import ScriptRuntimeException # (1)!

def command(sender, label, args):
    return True

try:
  ps.command.registerCommand(command, 'testcommand')
except ScriptRuntimeException as e: # (2)!
  print("The command /testcommand is already registered.")
```

1.  Import the appropriate Java exception for use in a `try` `except` block later.
2.  Except the Java `RuntimeException` in the same way that a Python error would be excepted.

Python exceptions, such as `TypeError`, `ValueError`, and `IndexError`, can be caught as well, using the same `try` and `except` syntax.

## Script Logging

Like scripts themselves, a script's logger is self-contained. Each script has its own logger, which is an instance of [dev.magicmq.pyspigot.util.logging.ScriptLogger](https://javadocs.magicmq.dev/pyspigot/dev/magicmq/pyspigot/util/logging/ScriptLogger.html). PySpigot creates a new logger for each running script. A script's logger is automatically assigned to its global namespace under the variable name `logger`, for easy access within the script. To access your script's logger, simply use the `logger` variable. For example:

=== "Bukkit"

    ``` py linenums="1"
    import pyspigot as ps

    def task():
        print('Task executed')

    task = ps.scheduler.runTaskLater(task, 100) # (1)!

    logger.info('Registered a new task: ' + str(task)) # (2)!
    ```

    1.  The default time unit is server ticks (1 second = 20 ticks) when registering tasks on the Bukkit platform.
    2.  The `logger` varaible can be utilized immediately without assigning it first. It is automatically assigned internally when the script is started.

=== "Velocity"

    ``` py linenums="1"
    import pyspigot as ps

    def task():
        print('Task executed')

    task = ps.scheduler.runTaskLaterAsync(task, 5000) # (1)!

    logger.info('Registered a new task: ' + str(task)) # (2)!
    ```

    1.  The default time unit is milliseconds (1 second = 1000 ms) when registering tasks on the Velocity platform. Additionally, synchronous tasks are not supported on Velocity, which is why `runTaskLaterAsync` is used here.
    2.  The `logger` varaible can be utilized immediately without assigning it first. It is automatically assigned internally when the script is started.

=== "BungeeCord"

    ``` py linenums="1"
    import pyspigot as ps

    def task():
        print('Task executed')

    task = ps.scheduler.runTaskLaterAsync(task, 5000) # (1)!

    logger.info('Registered a new task: ' + str(task)) # (2)!
    ```

    1.  The default time unit is milliseconds (1 second = 1000 ms) when registering tasks on the BungeeCord platform. Additionally, synchronous tasks are not supported on BungeeCord, which is why `runTaskLaterAsync` is used here.
    2.  The `logger` varaible can be utilized immediately without assigning it first. It is automatically assigned internally when the script is started.

When accessing your script's logger, you can use any of the functions listed [here](https://javadocs.magicmq.dev/pyspigot/dev/magicmq/pyspigot/util/logging/ScriptLogger.html).

???+ notice

    As of PySpigot 0.7.0, any messages printed via Python's `print` function are automatically captured and redirected to a script's logger. This makes it much easier to discern which print statements came from which script (if you have multiple scripts running), since the script's name is attached to all print messages.
    
    `[STDOUT]` is also attached to print messages to indicate that the message was printed to `STDOUT`.

    `[STDERR]` is attached to any error messages, to indicate that the message was printed to `STDERR`.

### Script Log Files

As stated previously, each script has its own log file. Script log files can be found in the `logs` folder within the PySpigot plugin folder. All messages related to a script will be logged in its respective log file. If you would like to disable file logging for a script, set the [`file-logging-enabled` script option](scriptoptions.md/#file-logging-enabled) to `false`.

The script logger also logs messages with an attached level. These levels are [Java logging levels](https://docs.oracle.com/en/java/javase/21/docs/api/java.logging/java/util/logging/Level.html). In short, the levels represent the acuity/severity of the log message. The three most common logging levels you will see are `INFO`, `WARNING`, and `SEVERE`:

- `INFO`: This level demarcates the message as an informational message.
- `WARNING`: This level demarcates the log message as a potential problem, but no immediate action is required.
- `SEVERE`: This level demarcates the log message as a serious failure, exception, or error, that requires prompt action.

Each script logger has a default minimum level at which log messages are logged. By default, this value is set to `INFO`. With a minimum level set at `INFO`, messages at the `INFO` level and higher (`WARNING`, `SEVERE`) are logged, but messages with lower status than the `INFO` level (`CONFIG`, `FINE`, `FINER`, `FINEST`) are discarded (not logged).

You may change the minimum logging level for a script by setting the [`min-logging-level` script option](scriptoptions.md/#min-logging-level).

A formatted time stamp is also printed in each log message with the exact time that the message was logged, in the local machine's time zone. If you would like to customize the timestamp format, edit the [`log-timestamp-format` value in the `config.yml`](../pyspigot/pluginconfiguration.md/#log-timestamp-format). The format should follow the pattern outlined [here](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/format/DateTimeFormatter.html).

## Non-ASCII Characters in Script Files

Jython reads and compiles script files using ASCII encoding. This means that it won't recognize non-ASCII characters in the file. You may see a `SyntaxError` when you load a script with non-ASCII characters. Additionally, in Python 2, the `str` type is a collection of 8-bit characters. Consequently, all characters in the English alphabet (and some basic symbols) can be represented using these 8-bit characters, but special symbols and characters from non-Latin alphabets cannot. There are a couple ways to work around these two constraints:

### Workaround 1

Jython allows you to specify the encoding of your script file. This is done by specifying an [encoding declaration](https://docs.python.org/2/reference/lexical_analysis.html#encoding-declarations) in your script. This ensures that when Jython reads and compiles the file, it will recognize the non-ASCII characters. Add the following to the *first or second line* of your script file:

`#coding: utf-8`

Of course, you can replace `utf-8` with whatever character encoding standard you'd like Jython to use. For a list of supported encoding schemes, see [this page](https://docs.python.org/2/library/codecs.html#standard-encodings).

???+ warning

    You must put the encoding declaration on either the first or second line of your script, as Jython only searches for encoding declarations in this area.

Python 2 includes a `unicode` type, which supports all UTF-8 characters (symbols, non-Latin alphabets, etc.). You can specify that you want to use the `unicode` type for the string (and not `str`) by adding a preceding `u`. For example:

=== "Bukkit"

    ``` py linenums="1"
    #coding: utf-8

    from org.bukkit import Bukkit

    Bukkit.broadcastMessage(u'Привет, мир!') # (1)!
    ```

    1.  Notice that a `u` directly preceeds the string. This denotes that the string should be treated a special `unicode` string, not a simple `str`.

=== "Velocity"

    ``` py linenums="1"
    #coding: utf-8

    from dev.magicmq.pyspigot.velocity import PyVelocity
    from net.kyori.adventure.text import Component

    proxy = PyVelocity.get().getProxy()

    proxy.sendMessage(Component.text(u'Привет, мир!')) # (1)!
    ```

    1.  Notice that a `u` directly preceeds the string. This denotes that the string should be treated a special `unicode` string, not a simple `str`.

=== "BungeeCord"

    ``` py linenums="1"
    #coding: utf-8

    from net.md_5.bungee.api import ProxyServer

    proxy = ProxyServer.getInstance()

    proxy.broadcast(u'Привет, мир!') # (1)!
    ```

    1.  Notice that a `u` directly preceeds the string. This denotes that the string should be treated a special `unicode` string, not a simple `str`.

### Workaround 2

You can use also use the hex codes of the unicode characters you want to write. You will still need to denote that the string is a unicode string by adding a preceding `u`, but you don't need to add an encoding declaration at the top of your script file since you aren't actually writing any non-ASCII characters in the file. For example:

=== "Bukkit"

    ``` py linenums="1"
    from org.bukkit import Bukkit

    Bukkit.broadcastMessage(u'\u041f\u0440\u0438\u0432\u0435\u0442, \u043c\u0438\u0440!')
    ```

=== "Velocity"

    ``` py linenums="1"
    from dev.magicmq.pyspigot.velocity import PyVelocity
    from net.kyori.adventure.text import Component

    proxy = PyVelocity.get().getProxy()

    proxy.sendMessage(Component.text(u'\u041f\u0440\u0438\u0432\u0435\u0442, \u043c\u0438\u0440!'))
    ```

=== "BungeeCord"

    ``` py linenums="1"
    from net.md_5.bungee.api import ProxyServer

    proxy = ProxyServer.getInstance()

    proxy.broadcast(u'\u041f\u0440\u0438\u0432\u0435\u0442, \u043c\u0438\u0440!')
    ```