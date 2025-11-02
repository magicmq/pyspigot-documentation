# PySpigot's Managers

PySpigot provides a variety of managers to more easily work with parts of the Bukkit, Velocity, and BungeeCord APIs, as well as other plugins, databases, and more. For instructions on importing these into your script, see below.

## Available Managers

The core managers of PySpigot are available on every platform and include the following:

- The [Script Manager](core/scripts.md), for loading, unloading, and otherwise managing scripts from within another script.
- The [Listener Manager](core/eventlisteners.md), for registering event listeners.
- The [Command Manager](core/commands.md), for registering and working with commands.
- The [Task Manager](core/tasks.md), for registering tasks with a platform's scheduler service.
- The [Config Manager](core/configuration.md), for working with configuration files.
- The [Database Manager](core/databases.md), to connect to and interact with SQL-type and Mongo databases.
- The [Redis Manager](core/redis.md), to connect to and interact with a Redis server instance.

The Bukkit platform has some additional managers:

- The [Plugin Messaging Manager](bukkit/messaging.md), to send and receive [plugin messages](https://docs.papermc.io/paper/dev/plugin-messaging/).
- The [ProtocolLib Manager](bukkit/protocollib.md), to work with ProtocolLib. This manager is optional, meaning it won't work unless if ProtocolLib is running on the server.
- The [Placeholder Manager](bukkit/placeholders.md), to work with PlaceholderAPI. This manager is optional, meaning it won't work unless if PlaceholderAPI is running on the server.

The BungeeCord platform has one additional manager:

- The [Protocolize Manager](bungee/protocolize.md), to work with Protocolize. This manager is optional, meaning it won't work unless if Protocolize is running on the server.

## Import Overview

Managers must be imported into your script in order for you to access them. The following table summarizes how to access managers, but read the sections below for more detail on how to import them:

| Manager                  | Access Via Helper Module         | Standalone Import                                                                  |
| ------------------------ | -------------------------------- | ---------------------------------------------------------------------------------- |
| Script Manager           | `pyspigot.script_manager()`      | `from dev.magicmq.pyspigot.manager.script import ScriptManager`                    |
| Listener Manager         | `pyspigot.listener_manager()`    | `from dev.magicmq.pyspigot.manager.listener import ListenerManager`                |
| Command Manager          | `pyspigot.command_manager()`     | `from dev.magicmq.pyspigot.manager.command import CommandManager`                  |
| Task Manager             | `pyspigot.task_manager()`        | `from dev.magicmq.pyspigot.manager.task import TaskManager`                        |
| Config Manager           | `pyspigot.config_manager()`      | `from dev.magicmq.pyspigot.manager.config import ConfigManager`                    |
| Database Manager         | `pyspigot.database_manager()`    | `from dev.nagicmq.pyspigot.manager.database import DatabaseManager`                |
| Redis Manager            | `pyspigot.redis_manager()`       | `from dev.magicmq.pyspigot.manager.redis import RedisManager`                      |
| Plugin Messaging Manager | `pyspigot.messaging_manager()`   | `from dev.magicmq.pyspigot.bukkit.manager.messaging import PluginMessagingManager` |
| ProtocolLib Manager^*^   | `pyspigot.protocol_manager()`    | `from dev.magicmq.pyspigot.bukkit.manager.protocol import ProtocolManager`         |
| Placeholder Manager^*^   | `pyspigot.placeholder_manager()` | `from dev.magicmq.pyspigot.bukkit.manager.placeholder import PlaceholderManager`   |
| Protocolize Manager^*^   | `pyspigot.protocol_manager()`    | `from dev.magicmq.pyspigot.bungee.manager.protocol import ProtocolManager`         |

\* Denotes an *optional* manager, meaning it will not work unless if the plugin it interfaces with is running on the server.

## Using PySpigot's Managers

To utilize these managers, they must be imported into your script. This can be done in three ways:

### Import managers via the `pyspigot.py` helper library

PySpigot ships with a `pyspigot.py` helper module, which is included in the PySpigot JAR file. This module contains several helper functions that makes it easier to access all the managers.

``` py linenums="1"
import pyspigot as ps

ps.script_manager().<function>
ps.listener_manager().<function>
ps.command_manager().<function>
ps.task_manager().<function>
ps.config_manager().<function>
ps.database_manager().<function>
ps.redis_manager().<function>
ps.messaging_manager().<function> # (1)!
ps.protocol_manager().<function> # (2)!
ps.placeholder_manager().<function> # (3)!
```

1.  Available on the Bukkit platform only.
2.  Available on the Bukkit platform only.
3.  Available on the Bukkit platform only.

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

For more information and documentation on the `pyspigot.py` helper module, see the [PySpigot Helper Module](../scripts/helpermodule.md) page.

### Import each manager individually

You can also import each manager class individually, directly from PySpigot's Java code.

``` py linenums="1"
from dev.magicmq.pyspigot.manager.script import ScriptManager as script
from dev.magicmq.pyspigot.manager.listener import ListenerManager as listener
from dev.magicmq.pyspigot.manager.command import CommandManager as command
from dev.magicmq.pyspigot.manager.task import TaskManager as scheduler
from dev.magicmq.pyspigot.manager.config import ConfigManager as config
...

script.get().<function>
listener.get().<function>
command.get().<function>
...
```

???+ warning

    If importing a manager individually, `get()` *must* be used each time the manager is called!