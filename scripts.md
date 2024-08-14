# Script Manager

The script manager is, for lack of a better word, the brains of PySpigot. It contains the code that loads, runs, and unloads scripts. It also acts as a manager that oversees script commands, listeners, and tasks, among other things. Under normal circumstances, you shouldn't need to interact with the script manager, as all of this is done automatically in the background. However, the script manager is available to you to access from your script if you need to use it to load/run or unload a script from within another script.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the script manager into your script.

# Script Manager Usage

Using the script manager to load/run and unload scripts is relatively simple. There are three functions available to do this:

- `script.loadScript(name)`: Loads a script with the given name. Returns a `RunResult` which represents the outcome of loading the script. See the below section for more information on `RunResult`.
- `script.loadScript(path)`: Loads a script with the given file path. Returns a `RunResult` which represents the outcome of loading the script. See the below section for more information on `RunResult`.
- `script.unloadScript(name)`: Unloads a script with he given name. This function will return `True` if the script unloaded successfully, or `False` if the script did not unload successfully. Unsuccessful unloads are usually accompanied by an error message printed to the console and to the script's log file (if file logging is enabled for the script).

!> *Never* unload a script from within itself. This *will* lead to unexpected behavior and errors.

## Run Results

A `RunResult` represents the outcome of running a script. There are four possible outcomes:

- `RunResult.SUCCESS`: Running the script was successful, and it is currently running. You can safely assume that if this is the RunResult, then the script is running without errors.
- `RunResult.FAIL_DISABLED`: Running the script failed, because the script is disabled in its options (in `script_options.yml`).
- `RunResult.FAIL_PLUGIN_DEPENDENCY`: Running the script failed, because it depends on one or more *plugins* that are not running.
- `RunResult.FAIL_ERROR`: Running the script failed, because there was an error when it ran. This error will be printed to the server console as well as the script's log file (if file logging is enabled for the script).
- `RunResult.FAIL_DUPLICATE`: Running the script failed, because there was a script already running with the same name.
- `RunResult.FAIL_SCRIPT_NOT_FOUND`: Running the script failre, because a script was not found with the given name.

The `RunResult` class is accessed from `dev.magicmq.pyspigot.manager.script.RunResult`. 

# Code Examples

Let's look at some code that loads, runs and unloads a script:

## Loading and Running a Script

```python
import pyspigot as ps
from dev.magicmq.pyspigot.manager.script import RunResult

run_result = ps.script.loadScript('script.py')

if (run_result == RunResult.SUCCESS):
	print('The script is running!')
elif (run_result == RunResult.FAIL_ERROR):
	print('The script did not load due to an error!')
```

On line 1, we import PySpigot as `ps` to utilize the script manager (`script`).

On line 2, we import `RunResult` to utilize it later to determine the outcome of loading the script.

On line 4, we load the script with the name `script.py`. We assign the returned value to `loaded_script`, we assign the outcome of loading the script (`RunResult`) to `run_result`.

On line 6, we check if the outcome of loading the script was `RunResult.FAIL_ERROR` (which means the script did not load due to an error) and print a message to the console if this was the case.

On line 8, we check if the outcome of loading the script was `RunResult.SUCCESS` (which means the script loaded successfully and is currently running) and print a message to the console if this was the case.

## Unloading a Script

```python
import pyspigot as ps

unload_result = ps.script.unloadScript('script.py')

if (unload_result == True):
	print('The script unloaded successfully without errors!')
```

On line 1, we import PySpigot as `ps` to utilize the script manager (`script`).

On line 3, we unload the script with the name `script.py`. We assign the returned value (`True` or `False`, depending on outcome) to `unload_result`.

On line 5, we check if the outcome of unloading the script was `True` (which means the script unloaded successfully without errors), and enter the if statement if so.

!> Use *extreme* caution when loading/unloading scripts through other scripts. This is a powerful but potentially destructive feature that can lead to unexpected behavior and errors.

## To summarize: {docsify-ignore}

- Use the script manager to load/run and unload a script from within another script.
- You must use both `loadScript` and `runScript` to load and run a script, as loading and running are two separate operations.
- `loadScript` returns a script object that should be passed to `runScript`. It will return `None` if the script did not load successfully.
- `runScript` returns a `RunResult` which can be used to determine the outcome of running the script.
- `unloadScript` returns `True` or `False`, `True` if the script unloaded successfully, or `False` if it did not.
- *Never* attempt to unload a script from within itself. This will lead to bugs/errors.