# Defining Commands

PySpigot allows you to define, register, and unregister commands from within scripts. These commands function in the same way as any plugin-defined command would in-game.

For instructions on importing the command manager into your script, visit the [General Information](usage.md) page.

???+ info

    This is not a comprehensive guide to commands in Bukkit. For a more complete guide to commands, see Spigot's tutorial on commands [here](https://www.spigotmc.org/wiki/create-a-simple-command/).

## Command Manager Usage

The command manager makes several functions available to you to register and unregister commands, including:

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
- `unregisterCommand(name)`: Allows you to unregister a command from your script using its name.
- `unregisterCommand(command)`: Allows you to unregister a command from your script. Accepts the `ScriptCommand` object returned from any of the `registerCommand` functions.

See the below section for a description of the arguments for the `registerCommand` functions.

All the above `registerCommand` functions return a `ScriptCommand` object, which can be used to unregister the command later, if desired.

???+ tip

    You **do not** need to unregister your commands when your script is stopped/unloaded. PySpigot will handle this for you.

### Function Arguments

The arguments of the above functions are described below:

- `command_function`: The function that should be called when the command is run in-game or in the server console. Command functions should always return `True` or `False`. Return `True` if the command was used correctly. Return `False` if it was not. This function should also accept three arguments:
    - `sender`, which represents the sender that executed the command (either a player or console),
    - `label`, which represents the label that was typed (either the command name or one of its aliases), and 
    - `args`, a list of arguments passed along with the command.
    - For more information, see the [Spigot tutorial on commands](https://www.spigotmc.org/wiki/create-a-simple-command/).
- `tab_function`: A function that should be called to generate a list of tab-completable items when typing the command.
- `name`: The name of the command being registered
- `description`: A description for the command
- `usage`: A message printed to the person executing the command if the usage is incorrect. This is controlled programatically by returning either `True` or `False` from the command function.
- `aliases`: A list of aliases for the command.
- `permission`: A permission node required to execute the command.

## Code Example

Let's look at the following code that defines and registers a command:

``` py linenums="1"
import pyspigot as ps # (1)!

def kick_command(sender, label, args): # (2)!
    #Do something...
    return True # (3)!

def tab_kick_command(sender, alias, args): # (4)!
    #Do something...
    return ['',] # (5)!

registered_command = ps.command.registerCommand(kick_command, tab_kick_command, 'kickplayer') # (6)!
```

1. Here, we import PySpigot as `ps` to utilize the command manager (`command`).

2. Here, we define a function called `kick_command` that takes three arguments, `sender`, `label`, and `args`. `sender` is who/what executed the command, `label` is exactly the command that was typed (if the command sender typed an alias, then this argument would be the alias that the command sender used). `args` is a list of `str` containing each argument that `sender` typed after the base command.

3. Here, we return a boolean value from the function. Command functions must always return either `True` or `False`. Whether or not you should return `True` or `False` is explained above.

4. Here, we define a function called `tab_kick_command` that takes the same parameters as `kick_command`. This function serves as the tab completer for the command. On line 9, we return a list of `str` from this function, which serves as the options that can be tab completed. Tab complete functions must return a list of `str`.

5. Here, we return a list from the function, representing the list of tab-completable arguments for the command. Tab completion functions must always return a list of `str` if there are tab-completable items or, if there are no tab-completable items, `None` or an empty list.

6. Here, we register the command with `registerCommand(command_function, tab_function, name)`. We also assign the returned value of `registerCommand` to `registered_command`. This is a `ScriptCommand` object, which represents the command that was registered. This can be used to unregister the command if you would like to do so later.

All commands must be registered with PySpigot's command manager to work. Commands can be registered in multiple ways. In the code above, the `registerCommand(function, name)` function is used, which takes two arguments:

- The first argument accepts the function that should be called when the command is executed.
- The second argument is the name of the command, a `str`.

???+ note:

    Command functions *must* return either `True` or `False`. Likewise, tab completion functions *must* return a list. If there are no tab complete suggestions to offer, return an empty list (`[]`).

    If a command/tab completion function does not return the correct type (or returns nothing at all), a warning message will be printed to the server console, but the command or tab completion will still complete successfully.

### Unregistering a Command

Continuing the above code example:

``` py linenums="1"
ps.command.unregisterCommand(registered_command) # (1)!
```

1. Here, we unregister the command by passing the `ScriptCommand` object we assigned earlier when registering the command. You can also unregister a command by passing its name if you don't want to store the `ScriptCommand` object.

???+ tip

    For more functional examples of commands, check out some of the [example scripts](../scripts/examples.md).

## Summary

- Commands are defined as functions in your script. Command functions *must* take three parameters: a sender, label, and args (the names of these parameters can be whatever you like). The function is called when the command is executed.
- Use a tab function to return tab-completable items when typing the command.
- All commands must be registered with PySpigot's command manager using `command.registerCommand(function, name)`.