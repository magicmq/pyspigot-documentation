# PySpigot Commands

PySpigot has several commands available to use, mostly for loading, unloading, and reloading scripts. They are all documented here.

???+ tip

    In general, when a command takes the name of a script or project, PySpigot uses the existence of a `.py` file extension in the name argument to determine if you are referencing a script or project:
    
    - When you are referencing a single-file script, use the name of the script file, **including the file extension** (for example, `test_script.py`). In addition, use the name of the file *only*. Do not include subfolders if the script is located within a subfolder. PySpigot automatically searches the `scripts` folder for a matching script file based on the name you provide.
        - For example, if you have a script with the path `scripts/test/test.py`, you should reference it by the file name only (`test.py`). To load it, you would run the command `/pyspigot load test.py`
    - When you are referencing a project, use the name of the project's folder (for example, `test_project`). Do not pass the main module of the project.

## Base Command

???+ note

    The base command of PySpigot differs depending on the platform. On Bukkit, the base command is `/pyspigot`, on BungeeCord, the base command is `/pybungee`, and on Velocity, the base command is `/pyvelocity`. Although the base commands differ, the permission nodes are kept the same across platforms (I.E. they start `pyspigot.` on all platforms). This was done to streamline permissions on proxy-wide permissions systems.

On Bukkit, the base command for PySpigot is `/pyspigot`. On BungeeCord, the base command for PySpigot is `/pybungee`. On Velocity, the base command for PySpigot is `/pyvelocity`.

=== "Bukkit"

    - Syntax: `/pyspigot <argument>`
    - Aliases: `ps`
    - Permission: `pyspigot.command.use`

=== "Velocity"

    - Syntax: `/pyvelocity <argument>`
    - Aliases: `pv`
    - Permission: `pyspigot.command.use`

=== "BungeeCord"

    - Syntax: `/pybungee <argument>`
    - Aliases: `pb`
    - Permission: `pyspigot.command.use`

If the user has the permission `pyspigot.command.listcmds`, running this command without any arguments will print a list of available commands.

All commands that follow are subcommands of the base command.

## Help Command

Displays several helpful links, including the documentation (this site), the PySpigot plugin page on Spigot, the official Discord, and a link to report issues/bugs on GitHub.

=== "Bukkit"

    - Syntax: `/pyspigot help`
    - Aliases: `gethelp`
    - Permission: `pyspigot.command.help`

=== "Velocity"

    - Syntax: `/pyvelocity help`
    - Aliases: `gethelp`
    - Permission: `pyspigot.command.help`

=== "BungeeCord"

    - Syntax: `/pybungee help`
    - Aliases: `gethelp`
    - Permission: `pyspigot.command.help`

## Info Command

Displays detailed information about a script or project, including uptime, registered listeners, commands, tasks, and more information. If getting info about a script, pass the script's name (including `.py`). If getting info about a project, pass the project's name (the name of the project folder). If `.py` is not included in the name, PySpigot will assume the name refers to a project.

=== "Bukkit"

    - Syntax: `/pyspigot info <scriptname/projectname>`
    - Aliases: `scriptinfo`
    - Permission: `pyspigot.command.info`

=== "Velocity"

    - Syntax: `/pyvelocity info <scriptname/projectname>`
    - Aliases: `scriptinfo`
    - Permission: `pyspigot.command.info`

=== "BungeeCord"

    - Syntax: `/pybungee info <scriptname/projectname>`
    - Aliases: `scriptinfo`
    - Permission: `pyspigot.command.info`

## ListScripts Command

Lists all scripts and projects, both unloaded and loaded. Loaded scripts and projects are shown in green; unloaded scripts and projects are shown in red. Use `[page]` to go to another page, if there are multiple pages available.

=== "Bukkit"

    - Syntax: `/pyspigot listscripts [page]`
    - Aliases: `list`, `scriptslist`, `ls`
    - Permission: `pyspigot.command.listscripts`

=== "Velocity"

    - Syntax: `/pyvelocity listscripts [page]`
    - Aliases: `list`, `scriptslist`, `ls`
    - Permission: `pyspigot.command.listscripts`

=== "BungeeCord"

    - Syntax: `/pybungee listscripts [page]`
    - Aliases: `list`, `scriptslist`, `ls`
    - Permission: `pyspigot.command.listscripts`

## Load Command

Loads and runs a script or project. If loading a script, pass the script's name (including `.py`). If loading a project, pass the project's name (the name of the project folder). If `.py` is not included in the name, PySpigot will assume the name refers to a project.

=== "Bukkit"

    - Syntax: `/pyspigot load <scriptname/projectname>`
    - Aliases: `start`
    - Permission: `pyspigot.command.load`

=== "Velocity"

    - Syntax: `/pyvelocity load <scriptname/projectname>`
    - Aliases: `start`
    - Permission: `pyspigot.command.load`

=== "BungeeCord"

    - Syntax: `/pybungee load <scriptname/projectname>`
    - Aliases: `start`
    - Permission: `pyspigot.command.load`

## LoadLibrary Command

Loads a Java library that you would like to use in a script. See [External Libraries](../scripts/externallibraries.md) for more information.

=== "Bukkit"

    - Syntax: `/pyspigot loadlibrary <filename>`
    - Aliases: `loadlib`
    - Permission: `pyspigot.command.loadlibrary`

=== "Velocity"

    - Syntax: `/pyvelocity loadlibrary <filename>`
    - Aliases: `loadlib`
    - Permission: `pyspigot.command.loadlibrary`

=== "BungeeCord"

    - Syntax: `/pyvelocity loadlibrary <filename>`
    - Aliases: `loadlib`
    - Permission: `pyspigot.command.loadlibrary`

## ReloadAll Command

Performs a complete reload of PySpigot, including the `config.yml`, `script_options.yml`, the library manager, and all scripts/projects.

=== "Bukkit"

    - Syntax: `/pyspigot reloadall`
    - Aliases: `reset`, `restart`, `reboot`, `resetall`
    - Permission: `pyspigot.command.reloadall`

=== "Velocity"

    - Syntax: `/pyvelocity reloadall`
    - Aliases: `reset`, `restart`, `reboot`, `resetall`
    - Permission: `pyspigot.command.reloadall`

=== "BungeeCord"

    - Syntax: `/pybungee reloadall`
    - Aliases: `reset`, `restart`, `reboot`, `resetall`
    - Permission: `pyspigot.command.reloadall`

## Reload Command

Reloads a script or project. Useful if you made changes to a script/project while it is running and you would like to reload it. If reloading a script, pass the script's name (including `.py`). If reloading a project, pass the project's name (the name of the project folder). If `.py` is not included in the name, PySpigot will assume the name refers to a project.

=== "Bukkit"

    - Syntax: `/pyspigot reload <scriptname/projectname>`
    - Aliases: None
    - Permission: `pyspigot.command.reload`

=== "Velocity"

    - Syntax: `/pyvelocity reload <scriptname/projectname>`
    - Aliases: None
    - Permission: `pyspigot.command.reload`

=== "BungeeCord"

    - Syntax: `/pybungee reload <scriptname/projectname>`
    - Aliases: None
    - Permission: `pyspigot.command.reload`

## ReloadConfig Command

Reloads PySpigot's `config.yml` and `script_options.yml`.

=== "Bukkit"

    - Syntax: `/pyspigot reloadconfig`
    - Aliases: `configreload`
    - Permission: `pyspigot.command.reloadconfig`

=== "Velocity"

    - Syntax: `/pyvelocity reloadconfig`
    - Aliases: `configreload`
    - Permission: `pyspigot.command.reloadconfig`

=== "BungeeCord"

    - Syntax: `/pybungee reloadconfig`
    - Aliases: `configreload`
    - Permission: `pyspigot.command.reloadconfig`

???+ warning

    The `/ps reloadconfig` command does not reload scripts or projects! Instead, use `/pyspigot reload <scriptname/projectname>` to reload a script/project, or `/ps reloadall` to reload all scripts and projects.

## Unload Command

Stops and unloads a script. If unloading a script, pass the script's name (including `.py`). If unloading a project, pass the project's name (the name of the project folder). If `.py` is not included in the name, PySpigot will assume the name refers to a project.

=== "Bukkit"

    - Syntax: `/pyspigot unload <scriptname/projectname>`
    - Aliases: `stop`
    - Permission: `pyspigot.command.unload`

=== "Velocity"

    - Syntax: `/pyvelocity unload <scriptname/projectname>`
    - Aliases: `stop`
    - Permission: `pyspigot.command.unload`

=== "BungeeCord"

    - Syntax: `/pybungee unload <scriptname/projectname>`
    - Aliases: `stop`
    - Permission: `pyspigot.command.unload`