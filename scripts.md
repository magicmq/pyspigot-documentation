# Script Manager

The script manager is, for lack of a better word, the brains of PySpigot. It contains the code that loads, runs, and unloads scripts. It also acts as a manager that oversees script commands, listeners, and tasks, among other things. Under normal circumstances, you shouldn't need to interact with the script manager, as all of this is done automatically in the background. However, the script manager is available to you to access from your script if you need to use it to load/run or unload a script from within another script.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the script manager into your script.

# Script Manager Usage

Using the script manager to load/run and unload scripts is relatively simple. There are three functions available to do this:

- `script.loadScript(name)`: Loads a script with the given name. Returns a script object that can later be passed to the `runScript` function to run the script. If there was an error when loading the script, this function will return `None`.
- `script.runScript(script)`: Runs the given script. Takes a script object, which is obtained from the `loadScript` function. Returns a `RunResult` which represents the outcome of loading the script. See 
- `script.unloadScript(name)`: Unloads a script witht he given name. This function will return `True` if the script unloaded successfully, or `False` if the script did not unload successfully. Unsuccessful unloads are usually accompanied by an error message printed to the console and the script's log file (if file logging is enabled for the script).

!> Loading and running a script are *separate* operations. If you want to run a script, you must load *and* run it with `loadScript` and `runScript`.

## Run Results

A `RunResult` represents the outcome of running a script. There are four possible outcomes:

- `RunResult.SUCCESS`: Running the script was successful, and it is currently running. You can safely assume that if this is the RunResult, then the script is running without errors.
- `RunResult.FAIL_DISABLED`: Running the script failed, because the script is disabled in its options (in `script_options.yml`).
- `RunResult.FAIL_DEPENDENCY`: Running the script failed, because it depends on one or more scripts that are not running.
- `RunResult.FAIL_ERROR`: Running the script failed, because there was an error when it ran. This error will be printed to the server console as well as the script's log file (if file logging is enabled for the script).

The `RunResult` class is accessed from `dev.magicmq.pyspigot.manager.script.RunResult`. 

# Code Examples

Let's look at some code that loads, runs and unloads a script:

## Loading and Running a Script

```python
from dev.magicmq.pyspigot import PySpigot as ps
from dev.magicmq.pyspigot.manager.script import RunResult

loaded_script = ps.script.loadScript('script.py')
run_result = ps.script.runScript(loaded_script)

if (run_result == RunResult.SUCCESS):
	print('The script is running!')
```

On line 1, we import PySpigot as `ps` to utilize the script manager (`script`).

On line 2, we import `RunResult` to utilize it later to determine the outcome of loading the script.

On line 4, we load the script with the name `script.py`. We assign the returned value to `loaded_script`, which represents the compiled version of the script, ready to be run.

On line 5, we run the script, passing the script object that we assigned on line 4. We assign the outcome of running the script, `RunResult`, to `run_result`.

On line 7, we check if the outcome of running the script was `RunResult.SUCCESS` (which means the script ran successfully and is currently running), and enter the if statement if this is true.

## Unloading a Script

```python
from dev.magicmq.pyspigot import PySpigot as ps

unload_result = ps.script.unloadScript('script.py')

if (unload_result == True):
	print('The script unloaded successfully without errors!')
```

On line 1, we import PySpigot as `ps` to utilize the script manager (`script`).

On line 3, we unload the script with the name `script.py`. We assign the returned value (`True` or `False`, depending on outcome) to `unload_result`.

On line 5, we check if the outcome of unloading the script was `True` (which means the script unloaded successfully without errors), and enter the if statement if so.

## To summarize: {docsify-ignore}

- Use the script manager to load/run and unload a script from within another script.
- You must use both `loadScript` and `runScript` to load and run a script, as loading and running are two separate operations.
- `loadScript` returns a script object that should be passed to `runScript`. It will return `None` if the script did not load successfully.
- `runScript` returns a `RunResult` which can be used to determine the outcome of running the script.
- `unloadScript` returns `True` or `False`, `True` if the script unloaded successfully, or `False` if it did not.