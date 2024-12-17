# Script Manager

The script manager serves as the de facto core of PySpigot, handling the loading, running, and unloading of scripts. It also oversees interactions between scripts and all the other managers within PySpigot. Typically, you won't need to interact directly with the script manager, as most operations are performed by the script manager automatically. However, if needed, you can access the script manager from within your script to load, run, or unload scripts programmatically.

For instructions on importing the script manager into your script, visit the [General Information](usage.md) page.

???+ warning

	Use *extreme* caution when loading/unloading scripts through other scripts. This is a powerful but potentially destructive feature that can lead to unexpected behavior and errors.

## Script Manager Usage

Using the script manager to load, run and unload scripts is relatively simple. There are three functions available to perform these operations:

- `script.loadScript(name)`: Loads a script with the given name. Returns a `RunResult` which represents the outcome of loading the script. See the below section for more information on `RunResult`.
- `script.loadScript(path)`: Loads a script with the given file path. Returns a `RunResult` which represents the outcome of loading the script. See the below section for more information on `RunResult`.
- `script.unloadScript(name)`: Unloads a script with he given name. This function will return `True` if the script unloaded successfully, or `False` if the script did not unload successfully. Unsuccessful unloads are usually accompanied by an error message printed to the console and to the script's log file (if file logging is enabled for the script).

In addition, the script manager contains server other useful functions:

- `script.isScriptRunning(name)`: Check if a script is running with the given name. Returns `True` or `False` accordingly.
- `script.getScriptPath(name)`: Gets the absolute path to a script with the given name. Useful if subfolders are being utilized within the `scripts` folder.
- `script.getScript(name)`: Get a running script. Returns a `Script` object if a running script was found with the given name, otherwise returns `None`. See the [Script Object](#the-script-object) section below for more information on the Script object.
- `script.getLoadedScripts()`: Returns a `Set` of all loaded/running scripts (as `Script` objects).
- `script.getLoadedScriptNames()`: Returns a `Set` of all loaded/running script names (as `str`s).
- `script.getAllScriptPaths()`: Returns a `Set`, sorted alphabetically, of all script paths (taking into account subfolders) within the `scripts` folder. 
- `script.getAllScriptNames()`: Returns a `Set`, sorted alphabetically, containing the names of all scripts within the `scripts` foler (taking into account subfolders).

???+ danger 

	***Never*** unload a script from within itself! This *will* lead to unexpected behavior and errors.

### Run Result

The `RunResult` is a value that represents the outcome of loading a script. Both of the `loadScript` functions return a `RunResult` representing the outcome of the load operation. There are four possible outcomes:

| RunResult                | Description                                                                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SUCCESS`                | The script loaded and ran successfully. The script is currently running.                                                                                      |
| `FAIL_DISABLED`          | Running the script failed because the script is disabled as per its script options (in `script_options.yml` or elsewhere)                                     |
| `FAIL_PLUGIN_DEPENDENCY` | Running the script failed because it depends on one or more plugins that are not running on the server.                                                       |
| `FAIL_ERROR`             | Running the script failed because there was an error when it ran. The error will be printed to the server console (and to the script's log file, if enabled). |
| `FAIL_DUPLICATE`         | Running the script failed because there is a script already running with the same name.                                                                       |
| `FAIL_SCRIPT_NOT_FOUND`  | Running the script failed because a script was not found in the `scripts` folder with the given name.                                                         |

The `RunResult` class is accessed by importing `dev.magicmq.pyspigot.manager.script.RunResult`.

### The Script Object

The `Script` object is the in-memory representation of a running script. It holds several key values related to a script, including its name, log file, options, and underlying Jython interpreter object. These can be accessed with the following functions:

- `getFile()`: Returns the `File` pertaining to the script.
- `getPath()`: Returns the absolute path of the script.
- `getName()`: Returns the name of the script.
- `getSimpleName()`: Returns the name of the script, without the extension (`.py`) at the end.
- `getOptions()`: Returns a `ScriptOptions` object representing the script's options. See the [JavaDocs](https://javadocs.magicmq.dev/pyspigot/dev/magicmq/pyspigot/manager/script/ScriptOptions.html) for a list of functions available in `ScriptOptions`.
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

- Use the script manager to load/run and unload a script from within another script.
- You must use both `loadScript` and `runScript` to load and run a script, as loading and running are two separate operations.
- `loadScript` returns a script object that should be passed to `runScript`. It will return `None` if the script did not load successfully.
- `runScript` returns a `RunResult` which can be used to determine the outcome of running the script.
- `unloadScript` returns `True` or `False`, `True` if the script unloaded successfully, or `False` if it did not.
- ***Never*** attempt to unload a script from within itself. This will lead to bugs/errors.