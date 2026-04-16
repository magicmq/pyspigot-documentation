# Defining Commands

PySpigot allows you to define, register, and unregister commands from within scripts. These commands function in the same way as any plugin-defined command would in-game.

For instructions on importing the command manager into your script, visit the [General Information](../usage.md) page.

???+ info

    This is not a comprehensive guide to commands in Bukkit, Velocity, or BungeeCord. For a more complete guide to commands, see the respective documentation for your platform:

    - [Bukkit](https://www.spigotmc.org/wiki/create-a-simple-command/)
    - [Velocity](https://docs.papermc.io/velocity/dev/command-api/)
    - [BungeeCord](https://www.spigotmc.org/wiki/creating-a-bungeecord-plugin/#commands)

## The `command` Decorator

PySpigot ships with a `decorators/command.py` helper module that provides Python **decorators** for registering commands. Using the decorator is the recommended way to register commands, as it is cleaner and more Pythonic than calling the command manager directly.

### Importing

Import the decorators from the module:

=== "Bukkit"

    ``` py
    from decorators.command import command
    from decorators.command import tab  # (1)!
    ```

    1. Only needed if you want to register a tab completion function using the `@tab` decorator. See [Tab Completion](#tab-completion) below.

=== "Velocity"

    ``` py
    from decorators.command import command
    from decorators.command import tab  # (1)!
    ```

    1. Only needed if you want to register a tab completion function using the `@tab` decorator. See [Tab Completion](#tab-completion) below.

=== "BungeeCord"

    ``` py
    from decorators.command import command
    from decorators.command import tab  # (1)!
    ```

    1. Only needed if you want to register a tab completion function using the `@tab` decorator. See [Tab Completion](#tab-completion) below.

### Basic Usage

Apply `@command('name')` to a function to register it as a command handler:

=== "Bukkit"

    ``` py linenums="1"
    from decorators.command import command

    @command('kickplayer') # (1)!
    def kick_command(sender, label, args): # (2)!
        # Do something...
        return True # (3)!
    ```

    1. Applying `@command('kickplayer')` registers `kick_command` as the handler for the `/kickplayer` command. No separate call to the command manager is needed.

    2. Command functions must accept three parameters: `sender` (who executed the command), `label` (the exact command or alias typed), and `args` (a list of arguments following the command name).

    3. Command functions must return `True` if the command was used correctly, or `False` if it was not. Returning `False` will display the usage message to the sender.

=== "Velocity"

    ``` py linenums="1"
    from decorators.command import command

    @command('kickplayer') # (1)!
    def kick_command(sender, label, args): # (2)!
        # Do something...
        return True # (3)!
    ```

    1. Applying `@command('kickplayer')` registers `kick_command` as the handler for the `/kickplayer` command. No separate call to the command manager is needed.

    2. Command functions must accept three parameters: `sender` (who executed the command), `label` (the exact command or alias typed), and `args` (a list of arguments following the command name).

    3. Command functions must return `True` if the command was used correctly, or `False` if it was not. Returning `False` will display the usage message to the sender.

=== "BungeeCord"

    ``` py linenums="1"
    from decorators.command import command

    @command('kickplayer') # (1)!
    def kick_command(sender, label, args): # (2)!
        # Do something...
        return True # (3)!
    ```

    1. Applying `@command('kickplayer')` registers `kick_command` as the handler for the `/kickplayer` command. No separate call to the command manager is needed.

    2. Command functions must accept three parameters: `sender` (who executed the command), `label` (the exact command or alias typed), and `args` (a list of arguments following the command name).

    3. Command functions must return `True` if the command was used correctly, or `False` if it was not. Returning `False` will display the usage message to the sender.

### Optional Parameters

The `@command` decorator accepts optional parameters to customize the command:

=== "Bukkit"

    - `name` *(required)* — the name of the command.
    - `description` — a description of the command. Defaults to `''`.
    - `usage` — a message shown to the sender when the command function returns `False`. Defaults to `''`.
    - `aliases` — a list of alternative names for the command. Defaults to `[]`.
    - `permission` — a permission node required to execute the command. Defaults to `None` (no permission required).

    ``` py linenums="1"
    from decorators.command import command

    @command('kickplayer', description='Kicks a player', usage='/kickplayer <player>', aliases=['kp', 'kick'], permission='myplugin.kick')
    def kick_command(sender, label, args):
        # Do something...
        return True
    ```

=== "Velocity"

    - `name` *(required)* — the name of the command.
    - `async_tab_complete` — whether the tab completion function runs asynchronously. Defaults to `True`.
    - `description` — a description of the command. Defaults to `''`.
    - `usage` — a message shown to the sender when the command function returns `False`. Defaults to `''`.
    - `aliases` — a list of alternative names for the command. Defaults to `[]`.
    - `permission` — a permission node required to execute the command. Defaults to `None` (no permission required).

    ``` py linenums="1"
    from decorators.command import command

    @command('kickplayer', description='Kicks a player', usage='/kickplayer <player>', aliases=['kp', 'kick'], permission='myplugin.kick')
    def kick_command(sender, label, args):
        # Do something...
        return True
    ```

=== "BungeeCord"

    - `name` *(required)* — the name of the command.
    - `description` — a description of the command. Defaults to `''`.
    - `usage` — a message shown to the sender when the command function returns `False`. Defaults to `''`.
    - `aliases` — a list of alternative names for the command. Defaults to `[]`.
    - `permission` — a permission node required to execute the command. Defaults to `None` (no permission required).

    ``` py linenums="1"
    from decorators.command import command

    @command('kickplayer', description='Kicks a player', usage='/kickplayer <player>', aliases=['kp', 'kick'], permission='myplugin.kick')
    def kick_command(sender, label, args):
        # Do something...
        return True
    ```

### Tab Completion

There are two ways to attach a tab completion function to a decorated command.

#### Using the `.tab` Method

When a function is decorated with `@command`, it gains a `.tab` method that registers a tab completion function for that command:

``` py linenums="1"
from decorators.command import command

@command('kickplayer')
def kick_command(sender, label, args):
    # Do something...
    return True

@kick_command.tab # (1)!
def tab_kick_command(sender, alias, args):
    return ['player1', 'player2'] # (2)!
```

1. Decorating `tab_kick_command` with `@kick_command.tab` registers it as the tab completion function for the `/kickplayer` command.

2. Tab completion functions must return a list of strings. Return an empty list (`[]`) or `None` if there are no suggestions to offer.

#### Using the `@tab` Decorator

Alternatively, the standalone `@tab('name')` decorator registers a tab completion function for a command that was already registered under a given name. This is useful when the command and its tab completer are defined in different places:

``` py linenums="1"
from decorators.command import command, tab

@command('kickplayer')
def kick_command(sender, label, args):
    # Do something...
    return True

@tab('kickplayer') # (1)!
def tab_kick_command(sender, alias, args):
    return ['player1', 'player2']
```

1. Decorating with `@tab('kickplayer')` registers `tab_kick_command` as the tab completion function for the command named `kickplayer`.

### Unregistering a Command

When a function is decorated with `@command`, two attributes are attached to it:

- `.registered_command` — the `ScriptCommand` object representing the registered command.
- `.unregister()` — a convenience method that unregisters the command.

``` py linenums="1"
kick_command.unregister() # (1)!
```

1. Unregisters the `kick_command` command. After this call, the command will no longer be available.

???+ tip

    You **do not** need to unregister your commands when your script is stopped/unloaded. PySpigot will handle this for you.

## Command Manager Usage

If you prefer to register commands manually without using the decorator, the command manager functions are available as an alternative. The functions available are:

- `registerCommand(command_function, name)`: The most basic way to register a command.
- `registerCommand(command_function, tab_function, name)`
- `registerCommand(command_function, name, permission)`
- `registerCommand(command_function, tab_function, name, permission)`
- `registerCommand(command_function, name, aliases, permission)`
- `registerCommand(command_function, tab_function, name, aliases, permission)`
- `registerCommand(command_function, name, description, usage)`
- `registerCommand(command_function, tab_function, name, description, usage)`
- `registerCommand(command_function, name, description, usage, aliases)`
- `registerCommand(command_function, tab_function, name, description, usage, aliases)`
- `registerCommand(command_function, tab_function, name, description, usage, aliases, permission)`: The most comprehensive way to register a command.
- `unregisterCommand(name)`: Unregisters a command by name.
- `unregisterCommand(command)`: Unregisters a command. Accepts the `ScriptCommand` object returned from any of the `registerCommand` functions.

All the above `registerCommand` functions return a `ScriptCommand` object, which can be used to unregister the command later, if desired.

### Function Arguments

The arguments of the above functions are described below:

- `command_function`: The function that should be called when the command is run. Command functions should always return `True` or `False`. Return `True` if the command was used correctly. Return `False` if it was not. This function should accept three arguments:
    - `sender` — the sender that executed the command (either a player or console),
    - `label` — the label that was typed (either the command name or one of its aliases), and
    - `args` — a list of arguments passed along with the command.
- `tab_function`: A function that should be called to generate a list of tab-completable items when typing the command. Must return a list of strings, or `None`/an empty list if there are no suggestions.
- `name`: The name of the command being registered.
- `description`: A description for the command.
- `usage`: A message printed to the sender if the command function returns `False`.
- `aliases`: A list of aliases for the command.
- `permission`: A permission node required to execute the command.

### Code Example

``` py linenums="1"
import pyspigot as ps

def kick_command(sender, label, args): # (1)!
    # Do something...
    return True # (2)!

def tab_kick_command(sender, alias, args): # (3)!
    return ['player1', 'player2'] # (4)!

registered_command = ps.command_manager().registerCommand(kick_command, tab_kick_command, 'kickplayer') # (5)!

# Later, to unregister:
ps.command_manager().unregisterCommand(registered_command) # (6)!
```

1. Define the command function. It accepts `sender`, `label`, and `args`.

2. Return `True` to indicate the command was used correctly. Return `False` to display the usage message.

3. Define the tab completion function. It accepts the same three parameters.

4. Return a list of strings as the tab-completable options. Return `None` or `[]` if there are no suggestions.

5. Register the command by passing the command function, tab function, and command name. The returned `ScriptCommand` is saved for later use.

6. Unregister the command by passing the `ScriptCommand` returned during registration.

???+ note

    Command functions *must* return either `True` or `False`. Likewise, tab completion functions *must* return a list. If there are no tab complete suggestions to offer, return an empty list (`[]`).

    If a command/tab completion function does not return the correct type (or returns nothing at all), a warning message will be printed to the server console, but the command or tab completion will still complete successfully.

???+ tip

    For more functional examples of commands, check out some of the [example scripts](../../scripts/examples.md).

## Summary

- Commands are defined as functions that accept three parameters: `sender`, `label`, and `args`. The function must return `True` or `False`.
- The recommended way to register a command is via the `@command('name')` decorator from `decorators/command.py`.
- Attach tab completion using the `.tab` method on the decorated function, or with the standalone `@tab('name')` decorator.
- Decorated functions gain a `.registered_command` attribute (the `ScriptCommand`) and an `.unregister()` method for easy cleanup.
- Alternatively, commands can be registered manually via the command manager using `registerCommand(function, name)` and related functions.
- You **do not** need to unregister commands when your script stops — PySpigot handles cleanup automatically.
