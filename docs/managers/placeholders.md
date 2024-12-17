# Registering PlaceholderAPI Placeholders

???+ warning

	The Placeholder manager is an *optional* manager. This manager should only be accessed if the PlaceholderAPI plugin is present on the server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with PlaceholderAPI if you would like to register placeholder expansions in your script.

For instructions on importing the placeholder manager into your script, visit the [General Information](usage.md) page.

## Placeholder Manager Usage

All placeholders created by scripts will follow this general format: `%script:<scriptname>_<placeholder>%`, where `<name>` is the name of your script (without .py), and `<placeholder>` is the specific placeholder, which you will handle yourself in a placeholder "replacer" function. See the code example below for details.

There are three functions available from the placeholder manager for registering/unregistering placeholders:

- `registerPlaceholder(placeholder_function)`: Registers a new placeholder expansion with default author ("Script Author") and version ("1.0.0"). When the placeholder is used, `placeholder_function` is called. It should return the text that should replace the placeholder. Returns a `ScriptPlaceholder`, which represents the placeholder that was registered.
- `registerPlaceholder(placeholder_function, author, version)`: Registers a new placeholder expansion with a custom author and version. Returns a `ScriptPlaceholder`, which represents the placeholder that was registered.
- `unregisterPlaceholder(placeholder)`: Unregisters a previously registered placeholder. Takes a ScriptPlaceholder that was previously returned by the `registerPlaceholder` function.

???+ tip

	You **do not** need to unregister your placeholders when your script is stopped/unloaded. PySpigot will handle this for you.

???+ note

	Scripts can only have one placeholder expansion registered at a time. To see how to define multiple placeholders for a script, see the code example below.

## Code Example

Let's look at the following code that defines and registers a placeholder expansion and replaces two placeholders:

``` py linenums="1"
import pyspigot as ps # (1)!

def replace(offline_player, placeholder): # (2)!
	if placeholder == 'placeholder1': # (3)!
		return 'Replace placeholder 1!'
	elif placeholder == 'placeholder2': # (4)!
		return 'Replace placeholder 2!'

placeholder = ps.placeholder.registerPlaceholder(replace) # (5)!
```

1. Here, we import PySpigot as `ps` to utilize the placeholder manager (`placeholder`).

2. Here, we define `replace`, a function that will be called when the placeholder is used. This function takes two parameters. `offline_player` is a Bukkit API OfflinePlayer that represents the player associated with the placeholder. `placeholder` is the text of the specific placeholder.

3. Here, we check if `placeholder` is equal to the specific placeholder `placeholder1`, a placeholder we define. If `placeholder` is equal to `placeholder1`, then we return the "replaced" text.

4. Here, we use `elif` to check if `placeholder` is equal to another specific placeholder `placeholder2`, another placeholder we define. If `placeholder` is equal to `placeholder2`, then we return the "replaced" text.

5. Here, we register our placeholder expansion with the `registerPlaceholder` function, passing the replacer function we defined earlier. We also assign the returned value of `registerPlaceholder` to `placeholder`. This is a `ScriptPlaceholder` object, which represents the placeholder expansion that was registered. This can be used to unregister the placeholder expansion if you would like to do so later.

Note that the replacer function for the placeholder expansion takes two arguments:

- The `offline_player` argument is the player associated with the placeholder. For example, if the placeholder is used in the context of a command, then `offline_player` would be the player that typed/executed the command. If no player was associated with the placeholder when it was used, then this argument will be `None`.
- The `placeholder` argument is the name of the placeholder that was used.

All placeholder expansion functions should follow this syntax.

???+ warning

	In the above example, and with all placeholders, `offline_player` could be `None` if there is no player associated with the placeholder.

To use the two placeholders we defined in the above example, we would use `%script:test_placeholder1%` and `%script:test_placeholder2%`.

### Unregistering a Placeholder

Continuing the above code example:

``` py linenums="1"
ps.placeholder.unregisterPlaceholder(placeholder) # (1)!
```

1. Here, we unregister the placeholder expansion by passing the `ScriptPlaceholder` object we assigned earlier when registering the placeholder.

### Multiple Placeholders

As you can see in the above example, you need not register a new placeholder expansion for each specific placeholder you want to define. Instead, all you need to do is check if the `placeholder` parameter of your replacer function is equal to the placeholder you want to define using `if` and `elif`.

## Summary

- Placeholders defined by scripts follow the format `%script:<scriptname>_<placeholder>%`. 
- Your placeholder replacer function should take two parameters, `offline_player` and `placeholder`. You can name them whatever you like. It will be called when the placeholder is used. `offline_player` is the player associated with the placeholder, if there is one. `placeholder` is the specific placeholder that was used.
- Register your placeholder with PySpigot's placeholder manager using `registerPlaceholder(placeholder_function)` or `registerPlaceholder(placeholder_function, author, version)`.
- Each script can only have one placeholder expansion registered at a time. Check the `placeholder` parameter of your replacer function (using `if` and `elif`) to handle multiple placeholders.
- When registering a placeholder expansion, the register functions all return a `ScriptPlaceholder`, which can be used to unregister the placeholder expansion at a later time.