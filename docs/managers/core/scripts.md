# Script Manager

The script manager serves as the de facto core of PySpigot, handling the loading, running, and unloading of scripts and projects. It also oversees interactions between scripts, projects, and all the other managers within PySpigot. Typically, you won't need to interact directly with the script manager, as most operations are performed internally. However, if needed, you can access the script manager from within your script to load, run, or unload scripts programmatically.

For instructions on importing the script manager into your script, visit the [General Information](../usage.md) page.

???+ warning

	Use *extreme* caution when loading/unloading scripts via other scripts. This is a powerful but potentially destructive feature that can lead to unexpected behavior and errors.

## Script Manager Usage

Using the script manager to load, run and unload scripts and projects is relatively simple. There are multiple functions available to perform these operations:

- `loadScript(name)`: Loads a script with the given name. Returns a `RunResult` which represents the outcome of loading the script. See the below section for more information on `RunResult`.
- `loadScript(path)`: Loads a script with the given file path. Returns a `RunResult` which represents the outcome of loading the script. See the below section for more information on `RunResult`.
- `loadProject(name)`: Loads a project with the given name. Returns a `RunResult` which represents the outcome of loading the project. See the below section for more information on `RunResult`.
- `loadProject(path)`: Loads a project with the given file path. Returns a `RunResult` which represents the outcome of loading the project. See the below section for more information on `RunResult`.
- `unloadScript(name)`: Unloads a script *or* project with he given name. This function will return `True` if the script unloaded successfully, or `False` if the script did not unload successfully. Unsuccessful unloads are usually accompanied by an error message printed to the console and to the script's log file (if file logging is enabled for the script).
- `unloadScript(script, error)`: Unloads a script *or* project by passing the `Script` object. This function will return `True` if the script/project unloaded successfully, or `False` if it did not. Unsuccessful unloads are usually accompanied by an error message printed to the console and to the script/project log file (if file logging is enabled for the script/project).
	- If the script/project is being unloaded as a result of an error, pass `True` for the `error` parameter. Otherwise, pass `False`. If `error` is `True`, the script/project's stop function will not be called.

In addition, the script manager contains server other useful functions:

- `isScriptRunning(name)`: Check if a script or project is running with the given name. Returns `True` or `False` accordingly.
- `getScriptPath(name)`: Gets the absolute path to a script with the given name. Useful if subfolders are being utilized within the `scripts` folder.
- `getProjectPath(name)`: Gets the absolute path to a project with the given name.
- `getScriptByName(name)`: Gets a running script or project by its name. Returns a `Script` object if a running script/project was found with the given name, otherwise returns `None`. See the [Script Object](#the-script-object) section below for more information on the `Script` object.
- `getLoadedScripts()`: Returns a `Set` of all running scripts and projects (as `Script` objects).
- `getLoadedScriptNames()`: Returns a `Set` of all running script and project names (as `str`s).
- `getAllScriptPaths()`: Returns a `Set`, sorted alphabetically, of all script paths within the `scripts` folder (taking into account subfolders). 
- `getAllProjectPaths()`: Returns a `Set`, sorted alphabetically, of all project paths within the `projects` folder. 
- `getAllScriptNames()`: Returns a `Set`, sorted alphabetically, containing the names of all scripts within the `scripts` folder (taking into account subfolders).
- `getAllProjectNames()`: Returns a `Set`, sorted alphabetically, containing the names of all projects within the `projects` folder.

???+ danger 

	***Never*** unload a script from within itself! This *will* lead to unexpected behavior and errors.

### Run Result

The `RunResult` is a value that represents the outcome of loading a script or project. The `loadScript` and `loadProject` functions return a `RunResult` representing the outcome of the load operation. There are four possible outcomes:

| RunResult                | Description                                                                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SUCCESS`                | The script loaded and ran successfully. The script is currently running.                                                                                      |
| `FAIL_DISABLED`          | Running the script failed because the script is disabled as per its script options (in `script_options.yml` or elsewhere)                                     |
| `FAIL_PLUGIN_DEPENDENCY` | Running the script failed because it depends on one or more plugins that are not running on the server.                                                       |
| `FAIL_ERROR`             | Running the script failed because there was an error when it ran. The error will be printed to the server console (and to the script's log file, if enabled). |
| `FAIL_DUPLICATE`         | Running the script failed because there is a script already running with the same name.                                                                       |
| `FAIL_SCRIPT_NOT_FOUND`  | Running the script failed because a script was not found in the `scripts` folder with the given name.                                                         |
| `FAIL_NO_MAIN`           | Running the project failed because a main script file was not found for the project.                                                                          |

The `RunResult` class is accessed by importing `dev.magicmq.pyspigot.manager.script.RunResult`.

### The Script Object

The `Script` object is the in-memory representation of a running script. It holds several key values related to a script, including its name, log file, options, and underlying Jython interpreter object. These can be accessed with the following functions:

- `getPath()`: Returns the absolute path of the script file, or folder, if the `Script` object represents a project.
- `getMainScriptPath()`: Returns the absolute path of the main script file for a multi-file project. If the `Script` object is a single-file script, this function returns the same value as `getPath()`.
- `getName()`: Returns the name of the script/project.
- `getSimpleName()`: Returns the name of the script, without the extension (`.py`) at the end.
- `getOptions()`: Returns a `ScriptOptions` object representing the script's options. See the [JavaDocs](https://javadocs.magicmq.dev/pyspigot/dev/magicmq/pyspigot/manager/script/ScriptOptions.html) for a list of functions available in `ScriptOptions`.
- `isProject()`: Returns `True` if the `Script` object is a multi-file project, `False` if it is not.
- `getModules()`: Returns a list of absolute paths corresponding to all modules belonging to the script/project. Useful for fetching all the modules of a multi-file project.
- `getStopFunctions()`: Returns a list of all stop functions belonging to the script/project.
- `addStopFunction(function)`: Adds a stop function to the script/project, to be called when the script unloads.
- `getInterpreter()`: Returns the script's underlying Jython interpreter object (`PythonInterpreter`).
- `getLogger()`: Returns the script's `ScriptLogger` object.
- `getLogFileName()`: Returns the script's log file name. This function always returns a file name for the script, even if file logging is disabled in its options.
- `getUptime()`: Returns the script's uptime in milliseconds.

???+ warning

	Again, exercise *extreme caution* when interacting with this portion of PySpigot, particularly the underlying Jython interpreter. Unexpected behavior can occur.

## Code Examples

Let's look at some code that loads, runs and unloads a script:

### Loading and Running a Script

``` py linenums="1"
import pyspigot as ps # (1)!
from dev.magicmq.pyspigot.manager.script import RunResult # (2)!

run_result = ps.script.loadScript('script.py') # (3)!

if (run_result == RunResult.SUCCESS): # (4)!
	print('The script is running!')
elif (run_result == RunResult.FAIL_ERROR): # (5)!
	print('The script did not load due to an error!')
```

1. Here, we import PySpigot as `ps` to utilize the script manager (`script`).

2. Here, we import `RunResult` to utilize it later to determine the outcome of loading the script.

3. Here, we load the script with the name `script.py`, and we assign the outcome of loading the script (the `RunResult`) to `run_result`.

4. Here, we check if the outcome of loading the script was `RunResult.SUCCESS` (which means the script loaded successfully and is currently running) and print a message to the console if this was the case.

5. Here, we check if the outcome of loading the script was `RunResult.FAIL_ERROR` (which means the script did not load due to an error) and print a message to the console if this was the case.

### Unloading a Script

``` py linenums="1"
import pyspigot as ps # (1)!

unload_result = ps.script.unloadScript('script.py') # (2)!

if (unload_result == True): # (3)!
	print('The script unloaded successfully without errors!')
```

1. Here, we import PySpigot as `ps` to utilize the script manager (`script`).

2. Here, we unload the script with the name `script.py`. We assign the returned value (`True` or `False`, depending on outcome) to `unload_result`.

3. Here, we check if the outcome of unloading the script was `True` (which means the script unloaded successfully without errors), and print a message to the console if this was the case.

## Summary

- Use the script manager to load/run and unload a script or project from within another script/project.
- Use the `loadScript` function to load a script. Use the `loadProject` function to load a project. Pass either the name of the script/project or a Path.
- The `loadScript` functions return a `RunResult`, which can be used to determine the outcome of running the script/project.
- The `Script` object is the in-memory representation of a running script or project. It holds several key values related to a script/project.
- `unloadScript` returns `True` or `False`, `True` if the script unloaded successfully, or `False` if it did not.
- ***Never*** attempt to unload a script from within itself. This will lead to bugs/errors.