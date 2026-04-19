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

1. `@placeholder()` registers `replace` as the placeholder replacer function with the default identifier (`script:<scriptname>`), author (`'Script Author'`), and version (`'1.0.0'`). Handle multiple placeholder names inside this single function using `if`/`elif`.

2. The replacer function receives two arguments: `offline_player` (the `OfflinePlayer` associated with the placeholder, or `None` if no player is associated) and `placeholder` (the specific placeholder text that was used).

3. Check the `placeholder` argument to decide what text to return for each specific placeholder.

### Optional Parameters

- `identifier` — the identifier for the placeholder expansion. Defaults to `None`, which uses `script:<scriptname>` (e.g. `script:test` for `test.py`). Invalid characters (`_`, `%`, `{`, `}`) are automatically stripped from custom identifiers.
- `author` — the author of the placeholder expansion. Defaults to `'Script Author'`.
- `version` — the version of the placeholder expansion. Defaults to `'1.0.0'`.

``` py linenums="1"
from decorators.placeholder import placeholder

@placeholder(identifier='myplaceholder', author='MyName', version='2.0.0') # (1)!
def replace(offline_player, placeholder):
    ...
```

1. With a custom identifier `myplaceholder`, the placeholder format becomes `%myplaceholder_<placeholder>%` instead of the default `%script:<scriptname>_<placeholder>%`.

### Relational Placeholders

There are two ways to attach a relational placeholder function to a decorated placeholder expansion.

#### Using the `.relational_function` Method

When a function is decorated with `@placeholder`, it gains a `.relational_function` method that registers a relational replacer for the same expansion:

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

1. Decorating `replace_relational` with `@replace.relational_function` registers it as the relational replacer for the same placeholder expansion. This is the simplest approach when both functions are defined together.

2. Relational replacer functions receive three arguments: `player_one`, `player_two` (the two players associated with the relational placeholder), and `placeholder` (the specific placeholder text used).

#### Using the `@relational_placeholder` Decorator

Alternatively, the `@relational_placeholder(identifier)` decorator factory registers a function as the relational replacer for a specific placeholder expansion by its identifier. This is useful when the relational function is defined separately or when working with a custom identifier:

``` py linenums="1"
from decorators.placeholder import relational_placeholder

@relational_placeholder('myplugin') # (1)!
def replace_relational(player_one, player_two, placeholder): # (2)!
    if placeholder == 'player_distance':
        return str(player_one.getLocation().distance(player_two.getLocation()))
```

1. `@relational_placeholder('myplugin')` registers `replace_relational` as the relational replacer for the placeholder expansion with identifier `myplugin`. Pass the same identifier that was used when registering the expansion with `@placeholder(identifier='myplugin')`. For the default identifier, pass `'script:<scriptname>'` (e.g. `'script:test'` for `test.py`).

2. Relational replacer functions receive three arguments: `player_one`, `player_two`, and `placeholder`.

### Unregistering a Placeholder

When a function is decorated with `@placeholder`, an `.unregister()` method is attached to it:

``` py linenums="1"
replace.unregister() # (1)!
```

1. Unregisters the placeholder expansion. After this call, none of the placeholders handled by `replace` will be resolved.

???+ tip

    You **do not** need to unregister your placeholders when your script is stopped/unloaded. PySpigot will handle this for you.

## Placeholder Manager Usage

By default, placeholders follow the format `%script:<scriptname>_<placeholder>%`, where `<scriptname>` is the name of your script (without `.py`) and `<placeholder>` is the specific placeholder text. With a custom identifier, the format is `%<identifier>_<placeholder>%`.

Relational placeholders follow the same pattern but are prefixed with `rel_`: `%rel_script:<scriptname>_<placeholder>%`, or `%rel_<identifier>_<placeholder>%` with a custom identifier.

???+ warning

    Custom identifiers must not contain the characters `_`, `%`, `{`, or `}`. These characters are automatically stripped from the identifier before registration. If any invalid characters are detected, a warning is logged. Attempting to register a second expansion with the same identifier (after stripping) as an existing expansion for the same script will raise a `ScriptRuntimeException`.

There are several functions available from the placeholder manager for registering and unregistering placeholders:

- `registerPlaceholder(placeholder_function)`: Registers a placeholder expansion with the default identifier, author (`"Script Author"`), and version (`"1.0.0"`). Returns a `ScriptPlaceholder`.
- `registerPlaceholder(placeholder_function, relational_placeholder_function)`: Same as above, but also registers a relational replacer function.
- `registerPlaceholder(placeholder_function, identifier, author, version)`: Registers a placeholder expansion with a custom identifier, author, and version. Pass `None` for `identifier` to use the default (`script:<scriptname>`). Returns a `ScriptPlaceholder`.
- `registerPlaceholder(placeholder_function, relational_placeholder_function, identifier, author, version)`: Same as above, but also registers a relational replacer function.
- `setRelationalPlaceholderFunction(relational_function, identifier)`: Sets the relational replacer function for a previously registered placeholder expansion identified by `identifier`.
- `unregisterPlaceholder(placeholder)`: Unregisters a placeholder expansion. Takes the `ScriptPlaceholder` returned by a register function.
- `unregisterPlaceholder(placeholder_function)`: Unregisters any placeholder expansion whose replacer function matches `placeholder_function`.

???+ tip

	You **do not** need to unregister your placeholders when your script is stopped/unloaded. PySpigot will handle this for you.

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

2. Here, `@placeholder()` registers `replace` as the placeholder replacer function for this script, using the default identifier.

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

2. Here, `@placeholder()` registers `replace` as the replacer for this script's placeholder expansion. A regular replacer function is required even if only relational placeholders are needed; in that case, you can simply `pass` with no logic.

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

You do not need to register a new placeholder expansion for each specific placeholder you want to define. Instead, check the `placeholder` parameter of your replacer function using `if` and `elif` to handle as many placeholders as needed within a single expansion.

## Summary

- By default, placeholders follow the format `%script:<scriptname>_<placeholder>%`. With a custom identifier, the format is `%<identifier>_<placeholder>%`. Invalid characters (`_`, `%`, `{`, `}`) are automatically stripped from custom identifiers; a warning is logged if any are found.
- Relational placeholders follow the same pattern prefixed with `rel_`: `%rel_script:<scriptname>_<placeholder>%` or `%rel_<identifier>_<placeholder>%`.
- Regular placeholder replacer functions should take two parameters, `offline_player` and `placeholder`. `offline_player` is the player associated with the placeholder (or `None`); `placeholder` is the specific placeholder text that was used.
- Relational placeholder replacer functions should take three parameters, `player_one`, `player_two`, and `placeholder`.
- The recommended way to register a placeholder is via the `@placeholder()` decorator from `decorators/placeholder.py`. Optional parameters: `identifier`, `author`, `version`.
- Attach a relational replacer using the `.relational_function` method on the decorated function, or with `@relational_placeholder(identifier)` for a specific expansion by identifier.
- Decorated functions gain an `.unregister()` method for easy cleanup.
- Alternatively, placeholders can be registered manually via the placeholder manager using `registerPlaceholder(placeholder_function, identifier, author, version)` and related overloads.
- You **do not** need to unregister placeholders when your script stops — PySpigot handles cleanup automatically.
