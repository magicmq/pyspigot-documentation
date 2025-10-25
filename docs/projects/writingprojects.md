# Writing Projects

As of PySpigot 0.9.0, PySpigot includes support for multi-file projects.

This page only covers key information and differences to know when writing multi-file projects as opposed to single-file scripts. For the full breadth of information on PySpigot scripts/projects, see the [General Information](../scripts/writingscripts.md) page.

## Basic Project Information

Writing a PySpigot project is nearly identical to writing a multi-module Python project. Modules within a project behave in the **exact same way** as they do in regular Python. For example, using a module would require it to be imported first, and the variables, functions, and scope are separate for each module. In addition, import mechanics also work the exact same way (I.E. an `__init__.py` file is required in a project subfolder to make it a Python package).

Like single-file scripts, PySpigot projects are designed to be *self-contained*. This means that each project is treated as a single "bundle", and all Python modules within the project's folder are *isolated* from files in another. Like single-file scripts, however, projects can interact with one another in various ways.

PySpigot projects are placed in the `projects` folder, which can be found in PySpigot's main plugin folder. Each project must be a folder; single-file projects are not allowed (as this would be a single-file script, which should be placed in the `scripts` folder instead). All folders within the `projects` folder are considered projects, and will be loaded automatically on server start/plugin load, unless if they are marked as disabled.

???+ warning

    Project names must be unique across other project names *and* single-file script names, as their names are used to identify them at runtime. For example, a project named `test_project` and a script named `test_project.py` are not compatible.

## Project Options

There are a variety of options that can be set for each project, including its main module, whether or not it is enabled, load priority, and logging options.

Project options are specified within the project's `project.yml` file, which is placed into the project's main directory. For example, a project named `test_project` would have its `project.yml` file at `/plugins/PySpigot/projects/test_project/project.yml`.

The most notable project option to be aware of is `main`, which specifies the main module of the project. PySpigot uses this value to determine which module to execute when the project is loaded. If this value isn't specified in a project's `project.yml` file, or the project doesn't have a `project.yml` file at all, then the default value under the `script-options-defaults` section of the main `config.yml` is used, (`main.py` by default).

All other project options are identical to the regular script options used for single-file scripts. For more information on project options, see the [project options page](projectoptions.md).

???+ notice

    Defining project options for a project is *optional*; projects will function normally without explicitly defining them (or creating a `project.yml` file at all), provided a module exists with the same name as the default `main` option in the `config.yml` (`main.py` by default).

### Project Permissions

???+ warning

    Project permissions are not supported on Velocity and BungeeCord, so configuring project permissions on these platforms has **no effect**.

**On the Bukkit platform only**, PySpigot allows projects to define a list of permissions that they use. This is useful if a project want to restrict access to certain features. Permissions are initialized and loaded just prior to parsing and executing the main module of the project, and are removed just after a project is stopped.

Project permissions are defined in the `project.yml` file of the project. For more information on how to define permissions, see the [documentation for project options](projectoptions.md#permissions).

## Project Loading

PySpigot loads and runs all projects in the projects folder automatically on plugin load or server start. Project load order is determined by load priority, as defined in the project's `project.yml` file. Projects that don't specify a load priority will inherit the default load priority specified in the `script-option-defaults` section of the `config.yml`. Projects that have the same load priority are loaded in alphabetical order.

Projects can also be manually loaded using the load command:

- On Bukkit: `/pyspigot load <projectname>`
- On Velocity: `/pyvelocity load <projectname>`
- On BungeeCord: `/pybungee load <projectname>`

 If you make changes to a project during runtime, the project must be reloaded for changes to take effect. Reload scripts with the reload command:

- On Bukkit: `/pyspigot reload <projectname>`
- On Velocity: `/pyvelocity reload <projectname>`
- On BungeeCord: `/pybungee reload <projectname>`

There is one config option related to loading projects:

- `script-load-delay`: This is the delay, in ticks, that PySpigot will wait **after server loading is completed** to load scripts and projects. There are 20 server ticks in one real-world second. For example, if the value is 20, then PySpigot will wait 20 ticks (or 1 second) after the server finishes loading to load scripts and projects. To disable the load delay, set this value to `0` or `-1`.

???+ notice

    Scripts and projects are interlaced when loading. In other words, they are loaded together. This means that the load priorities of scripts and projects are compared simultaneously. A project with a higher load priority would load earlier than a script with a lower load priority, and vice versa.

## Project Unloading

Projects can be manually unloaded using the unload command:

- On Bukkit: `/pyspigot unload <projectname>`
- On Velocity: `/pyvelocity unload <projectname>`
- On BungeeCord: `/pybungee unload <projectname>`

Additionally, running the reload command (`/pyspigot reload <projectname>` on Bukkit) will unload a project first before loading it again (if it was running beforehand).

### Unloading A Project from Within Itself

Unloading a project from within itself can be accomplished in the same way a running script would be terminated in a regular Python environment, via usage of the `sys.exit` function:

```py
import sys

sys.exit(0)
```

The `sys.exit` function can be safely called from any module within the project. Internally, calling `sys.exit` raises a `SystemExit` exception. PySpigot catches this exception and performs its standard unloading tasks to unload the project that raised the exception.

If you want to unload your project with a signal that an error occured, pass `1` to `sys.exit`. Doing so will prevent the `stop` function in the project's main module from being called on unload.

???+ warning

    *Do not* use the script manager to unload a project from within itself! This will lead to unexpected bugs/issues.

## Start and Stop Functions

The `start` and `stop` functions work in the exact same way for projects as they do for single-file scripts, with the important caveat that they **must be placed in the main module of your script**.

For more detailed information, see the [section on start and stop functions for single-file scripts](../scripts/writingscripts.md#start-and-stop-functions).

## The `pyspigot.py` Helper Module

PySpigot ships with a helper module called `pyspigot.py` that contains various useful functions to access PySpigot's manager classes. This helper module is accessible from any module within a project via a simple import:

``` py
import pyspigot as ps

...
```

For more information, see the [PySpigot Helper Module](../scripts/helpermodule.md) page.

## Global Variables

The global variables system functions in the exact same way for projects as it does for single-file scripts.

See the [Global Variables](../scripts/globalvariables.md) page for detailed information on how to use the global variables system.

## Project Errors and Exceptions

Errors and exceptions in projects function in the same way as they do for single-file scripts, except that for exceptions that occur in a module other than the main module of the project, the traceback will include the full call stack. The last line will be module that most directly caused/raised/threw the exception.

For defailed information, see the [section on script errors and exceptions for single-file scripts](../scripts/writingscripts.md/#script-errors-and-exceptions).

## Project Logging

Script logging for projects functions in the same way as it does for single-file scripts. For defailed information, see the [section on script logging for single-file scripts](../scripts/writingscripts.md/#script-logging).

## Non-ASCII Characters in Project Files

For information on how to include non-ASCII characters in project files, [visit this page](../scripts/writingscripts.md/#non-ascii-characters-in-script-files).