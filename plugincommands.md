# PySpigot Commands

PySpigot has several commands available to use, mostly for loading, unloading, and reloading scripts. They are all documented here.

The base command for PySpigot is `/pyspigot`.

- Syntax: `/pyspigot <argument>`
- Aliases: `ps`
- Permission: `pyspigot.command.listcmds`

## Help Command

Displays several helpful links, including the documentation (this site), the PySpigot plugin page on Spigot, the official Discord, and a link to report issues/bugs on GitHub.

- Syntax: `/pyspigot help`
- Aliases: `gethelp`, `info`, `getinfo`
- Permission: `pyspigot.command.help`

## ListScripts Command

Lists all scripts, both unloaded and loaded. Loaded scripts are shown in green, unloaded scripts are shown in red. Use `[page]` to go to another page, if there are multiple pages available.

- Syntax: `/pyspigot listscripts [page]`
- Aliases: `list`, `scriptslist`, `ls`
- Permission: `pyspigot.command.listscripts`

## Load Command

Loads and runs a script.

- Syntax: `/pyspigot load <scriptname>`
- Aliases: None
- Permission: `pyspigot.command.load`

## LoadLibrary Command

Loads a Java library that you would like to use in a script. See [External Libraries](librarymanager.md) for more information.

- Syntax: `/pyspigot loadlibrary <filename>`
- Aliases: `loadlib`
- Permission: `pyspigot.command.loadlibrary`

## ReloadAll Command

Performs a complete reload of PySpigot, including the `config.yml`, `script_options.yml`, the library manager, and all scripts.

- Syntax: `/pyspigot reloadall`
- Aliases: `reset`
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
- Aliases: None
- Permission: `pyspigot.command.unload`