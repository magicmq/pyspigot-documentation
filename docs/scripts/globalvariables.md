# Global Variables

PySpigot contains a global variable system that is available to scripts at runtime. On the Java side, this system relies on a [`HashMap`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html), which is a data structure that stores data in key:value pairs, much like a dict in Python. The intention of this system is to allow for shared variables across scripts. This feature might be useful if you are writing a system of scripts that rely on information from other scripts in order to function correctly.

Changes to variables inserted into this global set are automatically visible to all scripts. There is no need to re-insert a variable into the global set of variables if its value changes.

## Accessing the Global Variables System

PySpigot's global variables system is accessed just like any of PySpigot's managers. There are three ways to access it:

### Via the pyspgigot helper module

``` py linenums="1"
import pyspigot as ps

global_vars = ps.global_variables()

global_vars.<function>
```

The pyspigot helper module also defines a couple common aliases for convenience. Both `ps.global_vars` and `ps.gv` work. For example:

``` py linenums="1"
import pyspigot as ps

ps.global_vars.<function>
ps.gv.<function>
```

### Via the PySpigot class

``` py linenums="1"
from dev.magicmq.pyspigot import PySpigot as ps

global_vars = ps.global_vars

global_vars.<function>
```

### Via a direct import

``` py linenums="1"
from dev.magicmq.pyspigot.manager.script import GlobalVariables as global_vars

global_vars.get().<function>
```

???+ warning

	If you access the global variables system via a direct import, you must call `get()`!

## Usage

Basic usage of the global variables system would involve calling the `set`, `get`, and `remove` functions. See below for more detailed descriptions of these functions. Here is a short example:

=== "Script A"

	``` py linenums="1"
	import pyspigot as ps

	global_vars = ps.global_variables()# (1)!

	test = 'Global variable'

	global_vars.set('test', test)# (2)!
	```

	1. 	On this line, the global variables system is fetched from the `pyspigot` helper module.
	2.	On this line, a new value is set into the global variables system with the key "test", and the value of the variable `test` ("Global variable")

=== "Script B"

	``` py linenums="1"
	import pyspigot as ps

	global_vars = ps.global_variables()# (1)!

	test = global_vars.get('test')# (2)!

	print(test)

	global_vars.remove('test')# (3)!
	```

	1. 	On this line, the global variables system is fetched from the `pyspigot` helper module.
	2. 	On this line, the value assigned to the key "test" is obtained. This was set earlier in Script A.
	3.	Once finished obtaining the variable, it is removed with the `remove` function, passing the key that should be removed. Removing the variable once finished using it is crucial to prevent memory leaks. See the [Avoiding Memory Leaks](#avoiding-memory-leaks) section below for more information.

In the above code, we set the variable `test` in the global variables system in script A. Then, we retrieve and print the value of `test` from script B, and remove the variable after we retrieve it.

???+ warning

	Keys/Names are unique. If a new value is inserted into the set of global values with the same key as an existing value, then the old value will be overridden and inevitably lost.

??? notice

	There is no need to set the variable to the global variables system again if its value changes. Once a global variable is set in the global variables system, its value is updated automatically.

### Advanced Usage

The global variables system contains a function called `getHashMap()`, which returns the underlying Java HashMap where variables are stored. You can use this function to get the underlying HashMap in order to access more advanced functions. For a complete list of available functions within the Java HashMap class, see the [JavaDocs for HashMap](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html).

### Avoiding Memory Leaks

The global variables system is not smart. It will keep variables stored *indefinitely*, unless if they are overritten or intentionally deleted. The system will not remove variables unless it is explicitly told to do so. Consider the following example:

- Script A sets a variable to the global variables system for script B to use.
- Script B retrieves the variable and uses it.
- Script B is stopped or otherwise finishes using the variable.
- Script A continues going about its business.

In the above example, the global variable still exists in the global variables system, because neither script A nor script B cleared the variable from the system. In this case, the variable is leaky: it still exists and can be read, but it's not being used anymore. In programming, this is called a [memory leak](https://en.wikipedia.org/wiki/Memory_leak), and it is frowned upon because it leads to unnecessary use of memory. It's also preventable.

Returning to the above example: because script B is the script using the data, script A has no way of knowing when script B has seen that data. Therefore, it is script B's responsibility to remove the variable from the global variables system when it is finished retrieving it. You can see this is done in the above code once the `test` variable is printed

In general, when you're working with the global variables system, you should ensure that somewhere in your code, old variables are removed when they're no longer neeeded.

## Available Functions

The following is a list of available functions in the global variables system:

### `set(key, value)`

Insert a new value into the global set of variables with the given key (name). This function will always override an existing variable with the same key.

- **Parameters:**
	- key: The key of the global variable to set
	- value: The value of the variable to set
- **Returns:** The value that was previously set with the given key, or `None` if there was none

???+ warning

	The `set(key, value)` function will override an existing value with the same key!

### `set(key, value, override)`

Insert a new value into the global set of variables with the given key (name), with the option to override an existing value.

- **Parameters:**
	- key: The key of the global variable to set
	- value: The value of the variable to set
	- override: Pass `True` if an existing variable with the given key should be overridden. Pass `False` if an existing variable with the same key should *not* be overridden
- **Returns:** The value that was previously set with the given key, or `None` if there was none

### `remove(key)`

Remove a global variable with the given key (name).

- **Parameters:**
	- key: The key of the global variable to remove
- **Returns:** The value of the variable that was removed, or `None` if there was no global variable that was removed

### `get(key)`

Get a global variable with the given key (name).

- **Parameters:**
	- key: The key of the global variable to get
- **Returns:** The variable with the given key, or `None` if no values were found under the given key

### `getKeys()`

Get a list of all global variable keys (names) that are currently set.

- **Returns:** An immutable[^1] set of the keys of all global variables currently stored. Will return an empty set if there are no global variables that are set

[^1]: Immutable in this context means that any changes made to the set/list that is returned are not reflected in the global variables system. In other words, the set/list is not backed by the global variables HashMap.

### `getValues()`

Get a list of all global variable values that are currently set.

- **Returns:** An immutable[^1] set of the values of all currently stored global variables. Will return an empty set if there are no global variables

### `getHashMap()`

Get the underlying Java HashMap data structure that stores all global variables.

- **Returns:** The underlying HashMap, which is mutable[^2]

[^2]: Mutable in this context means that any changes made to the HashMap that is returned will be reflected in the global variables system.

### `contains(key)`

Check if a global variable is currently stored with the given key (name).

- **Returns:** `True` if there is a global variable with the given key, `False` if there is not

### `containsValue(value)`

Check if a global variable currently exists with the given value.

- **Returns:** `True` if there is a global variable with the given value, `False` if there is not

### `purge()`

Clear all variables from the global variables system. Acts as a hard "reset" to the system.

## Summary

- The global variables system allows for cross-script sharing of variables.
- The global variables system is accessed just like any of PySpigot's managers.
- Basic usage involves the `set`, `get`, and `remove` functions.
- The keys/names of global variables are unique. If a new variable is inserted with the same key as an existing variable, the old value will be lost.
- The values of variables stored in the global variables system are updated automatically, and changes are automatically visible to scripts accessing them without needing to call the `set` function over again.
- Global variables should be removed when scripts finish using them in order to avoid memory leaks.
- The global variables system uses a Java HashMap, and this underlying HashMap can be accessed with the `getHashMap` function.