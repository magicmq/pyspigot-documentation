# PySpigot Commands

PySpigot has several commands available to use, mostly for loading, unloading, and reloading scripts. They are all documented here.

???+ tip

    In general, when a command takes a script name, this is the name of the file *only*, including the extension (`.py`). This also holds true if the script file is located within a subfolder. PySpigot automatically searches the `scripts` folder for a matching script file based on the name you provide, so you don't need to specify the full path of the script file if the path contains subfolders.
    
    For example, if you have a script with the path `scripts/test/test.py`, you should reference it by the file name only (`test.py`). To load it, you would run the command `/pyspigot load test.py`.

## Base Command

The base command for PySpigot is `/pyspigot`. Running this command will print a list of available commands (as long as the user that typed the command has the permission `pyspigot.command.listcmds`).

- Syntax: `/pyspigot <argument>`
- Aliases: `ps`
- Permission: `pyspigot.command.listcmds`

All commands that follow are subcommands of the base command.

## Help Command

Displays several helpful links, including the documentation (this site), the PySpigot plugin page on Spigot, the official Discord, and a link to report issues/bugs on GitHub.

- Syntax: `/pyspigot help`
- Aliases: `gethelp`
- Permission: `pyspigot.command.help`

## Info Command

Displays detailed information about a script, including uptime, registered listeners, commands, tasks, and more information.

- Syntax: `/pyspigot info <scriptname>`
- Aliases: `scriptinfo`
- Permission: `pyspigot.command.info`

## ListScripts Command

Lists all scripts, both unloaded and loaded. Loaded scripts are shown in green, unloaded scripts are shown in red. Use `[page]` to go to another page, if there are multiple pages available.

- Syntax: `/pyspigot listscripts [page]`
- Aliases: `list`, `scriptslist`, `ls`
- Permission: `pyspigot.command.listscripts`

## Load Command

Loads and runs a script. Takes the name of the script only, even if the script resides in a subfolder.

- Syntax: `/pyspigot load <scriptname>`
- Aliases: `start`
- Permission: `pyspigot.command.load`

## LoadLibrary Command

Loads a Java library that you would like to use in a script. See [External Libraries](docs/scripts/externallibraries.md) for more information.

- Syntax: `/pyspigot loadlibrary <filename>`
- Aliases: `loadlib`
- Permission: `pyspigot.command.loadlibrary`

## ReloadAll Command

Performs a complete reload of PySpigot, including the `config.yml`, `script_options.yml`, the library manager, and all scripts.

- Syntax: `/pyspigot reloadall`
- Aliases: `reset`, `restart`, `reboot`, `resetall`
- Permission: `pyspigot.command.reloadall`

## Reload Command

Reloads a script. Useful if you made changes to a script while it is running and you would like to reload it.

- Syntax: `/pyspigot reload <scriptname>`
- Aliases: None
- Permission: `pyspigot.command.reload`

## ReloadConfig Command

Reloads PySpigot's `config.yml` and `script_options.yml`.

- Syntax: `/pyspigot reloadconfig`
- Aliases: `configreload`
- Permission: `pyspigot.command.reloadconfig`

???+ warning

    The `/ps reloadconfig` command does not reload scripts! Instead, use `/pyspigot reload <scriptname>` to reload a script, or `/ps reloadall` to reload all scripts.

## Unload Command

Stops and unloads a script.

- Syntax: `/pyspigot unload <scriptname>`
- Aliases: `stop`
- Permission: `pyspigot.command.unload`