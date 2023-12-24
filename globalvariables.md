# Global Variables

PySpigot contains a global variable system that is available to scripts at runtime. On the Java end, this system relies on a `HashMap`, which is a data structure that stores data in key:value pairs, much like a dict in Python. The intention of this system is to allow for shared variables across scripts. This feature might be useful if you are writing a system of scripts that rely on information from other scripts in order to function correctly.

# Accessing the Global Variables System

PySpigot's global variables system is accessed just like any of PySpigot's managers. There are three ways to access it:

```python
import pyspigot as ps

global_vars = ps.global_vars()

global_vars.<function>
```

Or:

```python
from dev.magicmq.pyspigot import PySpigot as ps

global_vars = ps.global_vars

global_vars.<function>
```

Or:

```python
from dev.magicmq.pyspigot.manager.script import GlobalVariables as global_vars

global_vars.get().<function>
```

# Usage

Basic usage of the global variables system would involve calling the `set`, `get`, and `remove` functions. See below for more detailed descriptions of these functions. Here is a short example:

Script A:
```python
import pyspigot as ps

global_vars = ps.global_vars()

test = 'Global variable'

global_vars.set('test', test)
```

Script B:
```python
import pyspigot as ps

global_vars = ps.global_vars()

test = global_vars.get('test')

print(test)

global_vars.remove('test')
```

In the above code, we set the variable `test` in the global variables system in script A. Then, we retrieve and print the value of `test` from script B, and remove the variable after we retrieve it.

?> Once a global variable is set in the global variables system, its value is updated automatically if it is changed, and all scripts will see this change. There is no need to set the variable to the system again if its value changes.

## Avoiding Memory Leaks

The global variables system is not smart. It will keep variables stored *indefinitely*, unless if they are overritten or intentionally cleared. The system will not remove variables unless it is explicitly told to do so. Consider the following example: script A sets a variable to the global variables system for script B to use. Script B retrieves the variable and uses it. Some time later, script B finishes its tasks and shuts down. But, the variable that script B used is still present in the global variables system. In this case, the variable is leaky: it still exists and can be read, but it's not being used anymore. In programming, this is called a [memory leak](https://en.wikipedia.org/wiki/Memory_leak), and it is frowned upon because it leads to unnecessary use of memory. It's also preventable.

Returning to the above example: because script B is the script using the data, script A has no way of knowing when script B has seen that data. Therefore, it is script B's responsibility to remove the variable from the global variables system when it is finished retrieving it. You can see this is done in the above code once the `test` variable is printed

In general, when you're working with the global variables system, you should ensure that somewhere in your code, old variables are removed when they're no longer neeeded.

## Advanced Usage

The global variables system contains a function called `getHashMap()`, which returns the underlying Java HashMap where variables are stored. You can use this function to get the underlying Hashmap and access more advanced functions. For a complete list of available functions within the Java HashMap class, see the [JavaDocs for HashMap](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html).

# Available Functions

The following is a list of available functions in the global variables system:

`set(key, value)`:
- Inserts a new value into the global set of variables with the given key (name). Will always override an existing variable with the same name.
- Parameters:
	- key: The name of the global variable to set
	- value: The value of the variable to set
- Returns: The value that was previously set with the given name, or `None` if there was none

!> The above function will override existing variables with the same name!

`set(key, value, override)`:
- Inserts a new value into the global set of variables with the given key (name), with the option to override an existing value.
- Parameters:
	- key: The name of the global variable to set
	- value: The value of the variable to set
	- override: `True` if an existing variable with the given name should be overriden, or `False` if it should not
- Returns: The value that was previously set with the given name, or `None` if there was none

`remove(key)`:
- Removes a global variable with the given key (name).
- Parameters:
	- key: The name of the global variable to remove
- Returns: The value of the variable that was removed, or null if there was none

`get(key)`:
- Gets a global variable with the given key (name).
- Returns: The variable with the given name, or null if there was none

`getKeys()`:
- Gets a list of all global variable keys (names) that are currently set.
- Returns: An immutable (changes are not reflected in the global variables system) set of the names of all global variables currently stored. Will return an empty set if there are no global variables

`getValues()`:
- Gets a list of all global variable values that are currently set.
- Returns: An immutable (changes are not reflected in the global variables system) set of the values of all currently stored global variables. Will return an empty set if there are no global variables

`getHashMap()`:
- Gets the underlying Java HashMap data structure that stores all global variables.
- Returns: The underlying HashMap, which is mutable (can make changes to it that will be reflected in the global variables system)

`contains(key)`:
- Check if a global variable is currently stored with the given key (name).
- Returns: `True` if there is a global variable with the given name, `False` if there is not

`containsValue(value)`:
- Check if a global variable is currently stored with the given value.
- Returns: `True` if there is a global variable with the given value, `False` if there is not

`purge()`:
- Clear all variables from the global variables system. Acts as a 'reset'.

## To summarize: {docsify-ignore}

- The global variables system allows for cross-script sharing of variables.
- The global variables system is accessed just like any of PySpigot's managers.
- Basic usage involves the `set`, `get`, and `remove` functions.
- The values of variables stored in the global variables system are updated automatically, and changes are automatically visible to scripts accessing them without needing to call the `set` function over again.
- Global variables should be removed when scripts finish using them in order to avoid memory leaks.
- The global variables system uses a Java HashMap, and this underlying HashMap can be accessed with the `getHashMap` function.