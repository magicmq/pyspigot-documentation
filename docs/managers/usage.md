# PySpigot's Managers

PySpigot provides a variety of managers to more easily work with parts of the Bukkit/Spigot API as well as other plugins, databases, and more. For instructions on importing these into your script, see below. The current managers built into PySpigot include:

- The [Script Manager](scripts.md), for loading and unloading scripts from within another script.
- The [Listener Manager](eventlisteners.md), for registering event listeners.
- The [Command Manager](commands.md), for registering and working with commands.
- The [Task Manager](tasks.md), for registering a variety of repeating, delayed, and asynchronous tasks.
- The [Config Manager](configuration.md), for working with configuration files.
- The [Database Manager](databases.md), to connect to and interact with SQL-type and Mongo databases.
- The [Redis Manager](redis.md), to connect to and interact with a Redis server instance.

Also included in PySpigot are two *optional* managers, which are available to use if the plugin they depend on is running on the server:

- The [ProtocolLib Manager](protocollib.md), to work with ProtocolLib.
- The [Placeholder Manager](placeholders.md), to work with PlaceholderAPI.

Managers must be imported into your script inn order for you to access them. The following table summarizes how to access managers, but read the sections below for more detail on how to import them:

| Manager             | Access Via Helper Module         | Access Under PySpigot       | Standalone Import                                                         |
| ------------------- | -------------------------------- | --------------------------- | ------------------------------------------------------------------------- |
| Script Manager      | `pyspigot.script_manager()`      | `PySpigot.script`           | `from dev.magicmq.pyspigot.manager.script import ScriptManager`           |
| Listener Manager    | `pyspigot.listener_manager()`    | `PySpigot.listener`         | `from dev.magicmq.pyspigot.manager.listener import ListenerManager`       |
| Command Manager     | `pyspigot.command_manager()`     | `PySpigot.command`          | `from dev.magicmq.pyspigot.manager.command import CommandManager`         |
| Task Manager        | `pyspigot.task_manager()`        | `PySpigot.scheduler`        | `from dev.magicmq.pyspigot.manager.task import TaskManager`               |
| Config Manager      | `pyspigot.config_manager()`      | `PySpigot.config`           | `from dev.magicmq.pyspigot.manager.config import ConfigManager`           |
| Database Manager    | `pyspigot.database_manager()`    | `PySpigot.database`         | `from dev.nagicmq.pyspigot.manager.database import DatabaseManager`       |
| Redis Manager       | `pyspigot.redis_manager()`       | `PySpigot.redis`            | `from dev.magicmq.pyspigot.manager.redis import RedisManager`             |
| Protocol Manager    | `pyspigot.protocol_manager()`    | `PySpigot.protocol`         | `from dev.magicmq.pyspigot.manager.protocol import ProtocolManager`       |
| Placeholder Manager | `pyspigot.placeholder_manager()` | `PySpigot.placeholder`      | `from dev.magicmq.pyspigot.manager.placeholder import PlaceholderManager` |

???+ warning

    The Protocol Manager and Placeholder Manager are *optional* managers. These managers should only be accessed if the ProtocolLib and/or PlaceholderAPI plugins are loaded and enabled.

## Using PySpigot's Managers

To utilize these managers, they must be imported into your script. This can be done in three ways:

### Import managers via the `pyspigot.py` helper library

As of version 0.5.0, PySpigot ships with a `pyspigot.py` helper module, which is automatically placed into the `python-libs` folder when the plugin is initialized. This module contains several functions to allow for access to all managers.

``` py linenums="1"
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

``` py linenums="1"
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

### Import all managers at once using the PySpigot class

This used to be the preferred way to access managers, but is no longer the preferred method as of version 0.5.0. This method is nevertheless still functional and can be used if desired.

``` py linenums="1"
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

### Import each manager individually

You can also import each manager class individually, directly from PySpigot's Java code.

``` py linenums="1"
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

???+ warning

    If importing a manager individually, `get()` *must* be used each time the manager is called!