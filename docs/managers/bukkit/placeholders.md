# Registering PlaceholderAPI Placeholders

???+ warning

	The Placeholder manager is an *optional* manager. This manager should only be accessed if the PlaceholderAPI plugin is present on the server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with PlaceholderAPI if you would like to register placeholder expansions in your script.

For instructions on importing the placeholder manager into your script, visit the [General Information](../usage.md) page.

## The `placeholder` Decorator

PySpigot ships with a `decorators/placeholder.py` helper module that provides Python **decorators** for registering placeholder expansions. Using the decorator is the recommended way to register placeholders, as it is cleaner and more Pythonic than calling the placeholder manager directly.

???+ warning

    The `decorators/placeholder` module checks whether PlaceholderAPI is available **at import time**. Importing from this module on a server where PlaceholderAPI is not installed will immediately raise a `ScriptRuntimeException`. Only import this module if you know PlaceholderAPI is present.

### Importing

``` py
from decorators.placeholder import placeholder
from decorators.placeholder import relational_placeholder  # (1)!
```

1. Only needed if you want to register a relational placeholder using the standalone decorator approach. See [Relational Placeholders](#relational-placeholders) below.

### Basic Usage

Apply `@placeholder()` to a function to register it as the placeholder replacer for your script:

``` py linenums="1"
from decorators.placeholder import placeholder

@placeholder() # (1)!
def replace(offline_player, placeholder): # (2)!
    if placeholder == 'placeholder1': # (3)!
        return 'Replace placeholder 1!'
    elif placeholder == 'placeholder2':
        return 'Replace placeholder 2!'
```

1. `@placeholder()` registers `replace` as the placeholder replacer function with default author (`'Script Author'`) and version (`'1.0.0'`). Since each script can only have one placeholder expansion registered at a time, handle multiple placeholder names inside this single function using `if`/`elif`.

2. The replacer function receives two arguments: `offline_player` (the `OfflinePlayer` associated with the placeholder, or `None` if no player is associated) and `placeholder` (the specific placeholder text that was used).

3. Check the `placeholder` argument to decide what text to return for each specific placeholder.

### Optional Parameters

- `author` — the author of the placeholder expansion. Defaults to `'Script Author'`.
- `version` — the version of the placeholder expansion. Defaults to `'1.0.0'`.

``` py linenums="1"
from decorators.placeholder import placeholder

@placeholder(author='MyName', version='2.0.0')
def replace(offline_player, placeholder):
    ...
```

### Relational Placeholders

There are two ways to attach a relational placeholder function to a decorated placeholder expansion.

#### Using the `.relational_function` Method

When a function is decorated with `@placeholder`, it gains a `.relational_function` method that registers a relational replacer function for the same placeholder expansion:

``` py linenums="1"
from decorators.placeholder import placeholder

@placeholder()
def replace(offline_player, placeholder):
    if placeholder == 'placeholder1':
        return 'Replace placeholder 1!'

@replace.relational_function # (1)!
def replace_relational(player_one, player_two, placeholder): # (2)!
    if placeholder == 'player_distance':
        return str(player_one.getLocation().distance(player_two.getLocation()))
```

1. Decorating `replace_relational` with `@replace.relational_function` registers it as the relational replacer for the same placeholder expansion.

2. Relational replacer functions receive three arguments: `player_one`, `player_two` (the two players associated with the relational placeholder), and `placeholder` (the specific placeholder text used).

#### Using the `@relational_placeholder` Decorator

Alternatively, the standalone `@relational_placeholder` decorator registers a function as the relational replacer for the script's placeholder expansion directly via the manager:

``` py linenums="1"
from decorators.placeholder import relational_placeholder

@relational_placeholder # (1)!
def replace_relational(player_one, player_two, placeholder):
    if placeholder == 'player_distance':
        return str(player_one.getLocation().distance(player_two.getLocation()))
```

1. Note: unlike `@placeholder()`, `@relational_placeholder` is not a factory — it is applied without parentheses.

### Unregistering a Placeholder

When a function is decorated with `@placeholder`, an `.unregister()` method is attached to it:

``` py linenums="1"
replace.unregister() # (1)!
```

1. Unregisters the entire placeholder expansion. After this call, none of the placeholders handled by `replace` will be resolved.

???+ tip

    You **do not** need to unregister your placeholders when your script is stopped/unloaded. PySpigot will handle this for you.

## Placeholder Manager Usage

All placeholders created by scripts will follow this general format: `%script:<scriptname>_<placeholder>%`, where `<scriptname>` is the name of your script (without .py), and `<placeholder>` is the specific placeholder, which you will handle yourself in a placeholder "replacer" function. See the code example below for details.

Relational placeholders can also be registered with the placeholder manager, and they follow the general format `%rel_script:<scriptname>_<placeholder>%`, where `<scriptname>` is the name of your script (without .py), and `<placeholder>` is the specific placeholder, which you will handle yourself in the relational placeholder "replacer" function. See the code example below for details.

There are several functions available from the placeholder manager for registering/unregistering placeholders:

- `registerPlaceholder(placeholder_function)`: Registers a new placeholder expansion with default author ("Script Author") and version ("1.0.0").
	- When the placeholder is used, `placeholder_function` is called. Should return the text that should replace the placeholder.
	- Returns a `ScriptPlaceholder`, which represents the placeholder that was registered.
- `registerPlaceholder(placeholder_function, relational_placeholder_function)`: Registers a new placeholder expansion with default author ("Script Author") and version ("1.0.0").
	- When the placeholder is used, `placeholder_function` is called. Should return the text that should replace the placeholder. This argument can be `None`.
	- When a relational placeholder is used, `relational_placeholder_function` is used. Should return the text that should replace the placeholder. This argument can be `None`.
	- Returns a `ScriptPlaceholder`, which represents the placeholder that was registered.
- `registerPlaceholder(placeholder_function, author, version)`: Registers a new placeholder expansion with a custom author and version.
	- When the placeholder is used, `placeholder_function` is called. Should return the text that should replace the placeholder.
	- Returns a `ScriptPlaceholder`, which represents the placeholder that was registered.
- `registerPlaceholder(placeholder_function, relational_placeholder_function, author, version)`: Registers a new placeholder expansion with a custom author and version.
	- When the placeholder is used, `placeholder_function` is called. Should return the text that should replace the placeholder.
	- When a relational placeholder is used, `relational_placeholder_function` is used. Should return the text that should replace the placeholder. This argument can be `None`.
	- Returns a `ScriptPlaceholder`, which represents the placeholder that was registered.
- `unregisterPlaceholder(placeholder)`: Unregisters a previously registered placeholder. Takes a ScriptPlaceholder that was previously returned by the `registerPlaceholder` function.

???+ tip

	You **do not** need to unregister your placeholders when your script is stopped/unloaded. PySpigot will handle this for you.

???+ note

	Scripts can only have one placeholder expansion registered at a time. To see how to define multiple placeholders for a script, see the code example below.

## Code Examples

### Regular Placeholder

Let's look at the following code that defines and registers a placeholder expansion and replaces two placeholders:

``` py linenums="1"
from decorators.placeholder import placeholder # (1)!

@placeholder() # (2)!
def replace(offline_player, placeholder): # (3)!
    if placeholder == 'placeholder1': # (4)!
        return 'Replace placeholder 1!'
    elif placeholder == 'placeholder2': # (5)!
        return 'Replace placeholder 2!'
```

1. Here, we import the `placeholder` decorator.

2. Here, `@placeholder()` registers `replace` as the placeholder replacer function for this script.

3. Here, we define `replace`. This function takes two parameters: `offline_player`, a Bukkit `OfflinePlayer` representing the player associated with the placeholder, and `placeholder`, the text of the specific placeholder that was used.

4. Here, we check if `placeholder` equals `placeholder1`. If so, we return the replacement text.

5. Here, we use `elif` to check for a second placeholder, `placeholder2`, and return its replacement text.

Note that the replacer function for the placeholder expansion takes two arguments:

- The `offline_player` argument is the player associated with the placeholder. For example, if the placeholder is used in the context of a command, then `offline_player` would be the player that typed/executed the command. If no player was associated with the placeholder when it was used, then this argument will be `None`.
- The `placeholder` argument is the name of the placeholder that was used.

All placeholder expansion functions should follow this syntax.

???+ warning

    In the above example, and with all placeholders, `offline_player` could be `None` if there is no player associated with the placeholder.

If the name of the script is `test.py`, the placeholders in the above example would be `%script:test_placeholder1%` and `%script:test_placeholder2%`.

### Relational Placeholder

Let's look at the following code that defines and registers a placeholder expansion for a relational placeholder.

``` py linenums="1"
from decorators.placeholder import placeholder # (1)!

@placeholder() # (2)!
def replace(offline_player, placeholder):
    pass  # No regular placeholders in this example

@replace.relational_function # (3)!
def replace_relational(player_one, player_two, placeholder): # (4)!
    if placeholder == 'player_distance': # (5)!
        return str(player_one.getLocation().distance(player_two.getLocation())) # (6)!
```

1. Here, we import the `placeholder` decorator.

2. Here, `@placeholder()` registers `replace` as the replacer for this script's placeholder expansion. A regular replacer function is required even if only relational placeholders are needed; in that case, you can simply pass with no logic.

3. Here, `@replace.relational_function` registers `replace_relational` as the relational replacer for the same placeholder expansion.

4. Here, we define `replace_relational`. This function takes three parameters: `player_one` and `player_two` (the two players associated with the relational placeholder), and `placeholder` (the specific relational placeholder text).

5. Here, we check if `placeholder` is equal to `player_distance`.

6. Here, we return the distance between the two players as the replacement text.

If the name of the script is `test.py`, the placeholder in the above example would be `%rel_script:test_player_distance%`.

???+ tip

	Multiple relational placeholders can be registered at the same time (using the same replacer function) by following the structure of the example in the previous section.

### Unregistering a Placeholder

Continuing the above code example:

``` py linenums="1"
replace.unregister() # (1)!
```

1. Here, we unregister the placeholder expansion using the `.unregister()` method attached by the decorator.

### Multiple Placeholders

As you can see in the above example, you need not register a new placeholder expansion for each specific placeholder you want to define. Instead, all you need to do is check if the `placeholder` parameter of your replacer function is equal to the placeholder you want to define using `if` and `elif`.

## Summary

- Placeholders defined by scripts follow the format `%script:<scriptname>_<placeholder>%`.
- Relational placeholders follow the format `%rel_script:<scriptname>_<placeholder>%`.
- Regular placeholder replacer functions should take two parameters, `offline_player` and `placeholder`. `offline_player` is the player associated with the placeholder (or `None`); `placeholder` is the specific placeholder text that was used.
- Relational placeholder replacer functions should take three parameters, `player_one`, `player_two`, and `placeholder`. `player_one` and `player_two` are the two players associated with the relational placeholder; `placeholder` is the specific placeholder text that was used.
- The recommended way to register a placeholder is via the `@placeholder()` decorator from `decorators/placeholder.py`.
- Attach a relational replacer to a decorated placeholder using the `.relational_function` method on the decorated function, or with the standalone `@relational_placeholder` decorator.
- Decorated functions gain an `.unregister()` method for easy cleanup.
- Alternatively, placeholders can be registered manually via the placeholder manager using `registerPlaceholder(placeholder_function, relational_placeholder_function)` or `registerPlaceholder(placeholder_function, relational_placeholder_function, author, version)`.
- Each script can only have one placeholder expansion registered at a time. Check the `placeholder` parameter of your replacer function (using `if` and `elif`) to handle multiple placeholders.
- You **do not** need to unregister placeholders when your script stops — PySpigot handles cleanup automatically.