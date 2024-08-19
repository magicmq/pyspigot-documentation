# PySpigot Commands

PySpigot has several commands available to use, mostly for loading, unloading, and reloading scripts. They are all documented here.

In general, when a command takes a script name, this is the name of the file *only*. This also holds true if the script file is located within a subfolder. For example, if you have a script with the path `scripts/test/test.py`, you should reference it by the file name only (`test.py`). To load it, you would run the command `/pyspigot load test.py`. PySpigot automatically searches the `scripts` folder for a matching script file based on the name you provide, so you don't need to specify the full path of the script file.

The base command for PySpigot is `/pyspigot`.

- Syntax: `/pyspigot <argument>`
- Aliases: `ps`
- Permission: `pyspigot.command.listcmds`

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

Loads and runs a script.

- Syntax: `/pyspigot load <scriptname>`
- Aliases: `start`
- Permission: `pyspigot.command.load`

## LoadLibrary Command

Loads a Java library that you would like to use in a script. See [External Libraries](librarymanager.md) for more information.

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

!> This does not reload scripts! Use `/pyspigot reload <scriptname>` to reload a script.

## Unload Command

Stops and unloads a script.

- Syntax: `/pyspigot unload <scriptname>`
- Aliases: `stop`
- Permission: `pyspigot.command.unload`