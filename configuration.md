# Configuration Files

With PySpigot, your scripts can load, access, and save to configuration files. All configuration files that scripts access using the config manager are automatically stored in the `config` folder located within PySpigot's plugin folder.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the config manager into your script.

This is not a comprehensive guide to working with config files. For more complete documentation on available methods/functions, see the [Javadocs](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/configuration/MemorySection.html). All methods listed here can be called from within your script.

# Config Manager Usage

The following functions are available from the config manager:

- `doesConfigExist(filePath)`: This checks if a config file exists under the given name or path. Returns `True` if the file exists, `False` if it does not.
- `loadConfig(filePath)`: This loads the config, and will create a file automatically if one does not already exist. Takes the name or path of the file you wish to load and/or create. Returns a `ScriptConfig` object representing the config that was loaded/created. No default values are specified.
- `loadConfig(filePath, defaults)`: This loads the config, and will create a file automatically if one does not already exist. Takes the name or path of the file you wish to load and/or create. Returns a `ScriptConfig` object representing the config that was loaded/created. Also takes a YAML-formatted string (`defaults`) that is used to specify the desired default values for the config.
- `reloadConfig(config)`: This reloads a config in case there any changes to the file that need to be loaded in. Takes the config (a `ScriptConfig` object) to reload. Returns another `ScriptConfig` object representing the config that was reloaded.
  - *Note:* The `reloadConfig` function will be removed in a future release of PySpigot. Instead, use the `reload` function within ScriptConfig (outlined in more detail below)
- `deleteConfig(filePath)`: This will delete the config file with the specified name or path. Returns `True` if a file was deleted, `False` if no file existed previously under the provided name or path.
- `getConfigFolder()`: This returns a Path to the folder (within the PySpigot plugin folder) where config files live.

?> The ConfigManager allows for usage of subfolders when working with script config files for organizational purposes. Simply pass the config file as a path to the above methods that accept a `filePath` parameter. For example, `ps.config_manager.loadConfig('test_script/config.yml')` will load the `config.yml` file in the `test_script` subfolder within the `configs` folder.

# ScriptConfig Usage

Loading/reloading a config returns a `ScriptConfig` object. This object has many methods/functions that you can use:

- Multiple functions are available for getting data from your config. For a complete list of methods/functions you can use to retrieve data from your config file, see the [Spigot JavaDocs](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/configuration/MemorySection.html)
- `set(key, value)`: Set a value in the config at the given key. Takes a key representing the key to write to and value which is the value to write.
- `getConfigFile()`: Get the file that corresponds to the config.
- `getConfigPath()`: Get the path for the file that corresponds to the config.
- `setDefaults(defaults)`: Used to set the default values for the config. Takes a YAML-formatted string.
- `load()`: Loads the configuration file. Under normal circumstances, this function should not be called, as it is done when `loadConfig` is called from the ScriptManager.
- `reload()`: Reloads the configuration from its file. Useful if changes where made to the file since the last load/reload, as these changes will not be reflected automatically until the config is reloaded.
- `save()`: This saves the config so that any values you set will be persistent. The configuration is automatically reloaded after saving.

!> Changes to the config are not saved to the file automatically! You *must* call `save` to write changes to the config file.

# Code Example

Let's take a look at the following code that loads a config, reads a number and string from it, writes to it, then saves it.

```python
import pyspigot as ps

script_config = ps.config.loadConfig('test.yml')

a_number = script_config.getInt('test-number')
a_string = script_config.getString('test-string')

script_config.set('test-set', 1337)
script_config.save()
```

On line 1, we import PySpigot as `ps` to utilize the config manager (`config`).

On line 3, we load the config using the config manager. Fortunately, the config manager is easily accessible from the `pyspigot` helper module under the variable name `config`, for easy access. The `loadConfig` function takes a string representing the name of the config file to load. If the file does not exist, it will create it automatically.

On lines 5 and 6, we read a number and a string from the config, respectively, by using `getInt` and `getString`.

Finally, on lines 7 and 8, we first set the value 1337 to a config key called `test-set`. Then, we save the config with `script_config.save()`.

!> Configuration files are not unique to each script! Any script can access any config file. Use names that are unique.

# Special Python Data Types

## Lists, Sets, and Tuples

Lists, sets, and tuples are all saved in yaml syntax as a list, which will look like this:

```yaml
list:
  - 'item1'
  - 'item2'
  - 'item3'
```

## Dictionaries

A dictionary is represented in the `key: value` format in yaml syntax. For example, consider the following code:

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}

script_config.set('dict', thisdict)
```

This will output to yaml syntax in the following format:

```yaml
dict:
  year: 1964
  model: Mustang
  brand: Ford
```

Of course, you may also nest lists, sets, tuples, and additional dictionaries within dictionaries and they will be saved accordingly. Dictionaries are particularly useful for setting multiple config values at the same time, without having to set each value individually.

## To summrize: {docsify-ignore}

- Scripts can load and save to config files that are automatically stored in PySpigot\'s plugin folder in the `configs` folder.
- To load a config, use `config.load(filePath)`. The `filePath` parameter is the name *or* path of the config file you wish to load (including the `.yml` extension). If the config file does not exist, it will be created for you automatically. This returns a `ScriptConfig` object that is used to access the contents of the config and write to the config.
- For all available functions/methods to get values from a loaded config, see the [Javadocs](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/configuration/MemorySection.html).
- To set a value in a config, use `script_config.set(key, value)`, where `key` is the key you wish to write to and `value` is the value to write.
- Finally, to save a config, use `script_config.save()`.
