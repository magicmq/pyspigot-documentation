# Placeholder Manager

!> The Placeholder manager is an *optional* manager. This manager should only be accessed if the PlaceholderAPI plugin is present on the server when the PySpigot plugin is enabled.

PySpigot includes a manager that interfaces with PlaceholderAPI if you would like to register placeholder expansions in your script.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the placeholder manager into your script.

# Placeholder Manager Usage

All placeholders created by scripts will follow this general format: `%script:<scriptname>_<placeholder>%`, where `<name>` is the name of your script (without .py), and `<placeholder>` is the specific placeholder, which you will handle yourself in a placeholder "replacer" function. See the code example below for details.

There are three functions available from the placeholder manager for registering/unregistering placeholders:

- `registerPlaceholder(placeholder_function)`: Registers a new placeholder expansion with default author ("Script Author") and version ("1.0.0"). When the placeholder is used, `placeholder_function` is called. It should return the text that should replace the placeholder. Returns a ScriptPlaceholder, which represents the placeholder that was registered.
- `registerPlaceholder(placeholder_function, author, version)`: Registers a new placeholder expansion with a custom name and version. Returns a ScriptPlaceholder, which represents the placeholder that was registered.
- `unregisterPlaceholder(placeholder)`: Unregisters a previously registered placeholder. Takes a ScriptPlaceholder that was previously returned by the `registerPlaceholder` function.

?> You **do not** need to unregister your placeholders when your script is stopped/unloaded. PySpigot will handle this for you.

!> Scripts can only have one placeholder expansion registered at a time. To see how to define multiple placeholders for a script, see the code example below.

# Code Example

Let's look at the following code that defines and registers a placeholder:

```python
import pyspigot as ps

def replace(offline_player, placeholder):
	if placeholder == 'placeholder1':
		return "Replace placeholder 1!"
	elif placeholder == 'placeholder2':
		return "Replace placeholder 2!"

placeholder = ps.placeholder.registerPlaceholder(replace)
```

On line 1, we import PySpigot as `ps` to utilize the placeholder manager (`placeholder`).

On line 3, we define `replace`, a function that will be called when the placeholder is used. This function has two parameters. `offline_player` is a Bukkit API OfflinePlayer that represents the player associated with the placeholder. `placeholder` is the text of the specific placeholder.

!> `offline_player` could be null if there is no player associated with the placeholder.

On line 4, we check if `placeholder` is equal to the specific placeholder we want to define, `placeholder1`. Then, on line 5, we return the "replaced" text.

On line 6, we use `else if` to define another specific placeholder, `placeholder2`. We return the text for this placeholder on line 6.

As you can see, you need not register a new placeholder expansion for each specific placeholder you want to define. Instead, all you need to do is check if the `placeholder` parameter of your replacer function is equal to the placeholder you want to define. Use `if` and `else if` statements for this.

If we call the script test.py, the two placeholders we define in the above code are `%script:test_placeholder1%` and `%script:test_placeholder2%`.

## To summarize: {docsify-ignore}

- Placeholders defined by scripts follow the format `%script:<scriptname>_<placeholder%`. 
- Your placeholder replacer function should take two parameters, `offline_player` and `placeholder`. You can name them whatever you like. It will be called when the placeholder is used. `offline_player` is the player associated with the placeholder, if there is one. `placeholder` is the specific placeholder that was used.
- Register your placeholder with PySpigot's placeholder manager using `registerPlaceholder(placeholder_function)` or `registerPlaceholder(placeholder_function, author, version)`.
- Each script only needs to have one placeholder expansion registered. Check the `placeholder` parameter of your replacer function to handle specific placeholders.
- When registering a placeholder expansion, the register functions all return a `ScriptPlaceholder`, which can be used to unregister the placeholder expansion at a later time.