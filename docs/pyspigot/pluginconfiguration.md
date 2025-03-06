# PySpigot Configuration

The following page provides information on all configuration values that can be found in PySpigot's `config.yml`.

## `metrics-enabled`

Specifies whether PySpigot will collect metrics data and submit this data to bStats. Set to `false` to disable collection of metrics data. You may also disable bStats server-wide in the bStats config file under `/plugins/bStats`.

``` yaml linenums="1"
metrics-enabled: true
```

Default: `true`

## `script-load-delay`

The delay, in ticks, that PySpigot will wait **after server loading is completed** to load scripts. There are 20 server ticks in one real-world second. For example, if the value is 20, then PySpigot will wait 20 game ticks (or 1 real-world second) after the server finishes loading to load scripts. Set to `-1` to disable a load delay and load scripts immediately.

``` yaml linenums="1"
script-load-delay: 20
```

Default: `20`

## `library-relocations`

List of relocation rules for external Java libraries in the `java-libs` folder. This feature is useful to relocate class names of an external Java library if a different version of the library is already present and in use by another plugin on the server. Format as `<pattern>|<relocated pattern>`.

``` yaml linenums="1"
library-relocations:
  - 'org.apache.commons|relocated.org.apache.commons'
```

Default: None (empty list)

## `log-timestamp-format`

The Date/time format that should be used to timestamp log messages when printing them to a script's log file. The timestamp should be in an appropriate [SimpleDateFormat](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/SimpleDateFormat.html).

``` yaml linenums="1"
log-timestamp-format: 'MMM dd yyyy HH:mm:ss'
```

Default: `MMM dd yyyy HH:mm:ss`

## `script-action-logging`

Specifies whether or not messages should be printed to console when a script is loaded, unloaded, and reloaded.

``` yaml linenums="1"
script-action-logging: true
```

Default: `true`

## `verbose-redis-logging`

Specifies whether all redis events should be logged to a script's logger, or if only crucial events should be logged. Crucial events include reconnect attempts and reconnect failures. If set to `false`, only these crucial events will be logged.

``` yaml linenums="1"
verbose-redis-logging: true
```

Default: `true`

## `script-unload-on-plugin-disable`

Specifies whether a script should be automatically unloaded if a plugin it depends on is unloaded. This value only applies to scripts that have plugin dependencies listed in the `script_options.yml` file. This feature is especially useful to ensure that when a script depends on a plugin, its shutdown tasks complete successfully if one or more of the plugins it depends on is/are disabled.

``` yaml linenums="1"
script-unload-on-plugin-disable: true
```

Default: `true`

## `jython-options`

This section allows you to specify options that pertain to Jython.

### `init-on-startup`

Specifies whether or not Jython should be initialized on plugin load. If this is set to `true`, script load times can be reduced by initializing Jython earlier (during server startup) rather than just prior to loading the first script.

``` yaml linenums="1"
jython-options:
  init-on-startup: true
```

Default: `true`

### `properties`

A list of properties that should be passed to Jython when it is initialized. By default, the `python.cachedir.skip` property is passed with a value of `true`. Unless you know what you are doing, this option should be left as-is, as under normal circumstances, embedded Jython should not utilize a cache directory.

See [this page](https://www.jython.org/registry.html) for a more detailed explanation as well as the [RegistryKey class](https://github.com/jython/jython/blob/master/src/org/python/core/RegistryKey.java) in the Jython source code for a complete list of Jython properties.

``` yaml linenums="1"
jython-options:
  properties:
    - 'python.cachedir.skip=true'
```

Default: List containing `python.cachedir.skip=true`

### `args`

A list of arguments that should be passed to Jython when it is initialized. This is equivalent to the `sys.argv` list in regular Python.

``` yaml linenums="1"
jython-options:
  args:
    - 'test_argument'
```

Default: None (empty list)

## `script-option-defaults`

The script option defaults are default [script options](../scripts/scriptoptions.md) that can be specified. The options defined in this section serve as fallback falues that should be used in the case that a script has one or more script options that aren't defined in the `script_options.yml` file.

### `enabled`

Used to enable or disable a script. To disable a script, set this value to `false`.

``` yaml linenums="1"
script-option-defaults:
  enabled: true
```

Default: `true`

### `load-priority`

Specifies an integer load priority for the script. Scripts are loaded in order from highest to lowest load priority. In other words, scripts that have a higher load priority are loaded earlier, and scripts with a lower load priority are loaded later. If multiple scripts have the same load priority, they are loaded in alphabetical order. To list a dependency, use the full name of the script (including `.py`).

``` yaml linenums="1"
script-option-defaults:
  load-priority: 1
```

Default: `1`

### `plugin-depend`

Specifies a list of plugins that the script depends on. The script will not load if any of the plugin dependencies are not loaded and running on the server. Additionally, if the [#script-unload-on-plugin-disable] parameter is set to `true`, any scripts that depend on that plugin as specified under this option are automatically unloaded when the plugin is unloaded/disabled.

??? note

    If you are working with ProtocolLib or PlaceholderAPI in your script, you **do not** need to specify either of them here. PySpigot has built-in support for these two plugins, and the dependency management is handled automatically.

``` yaml linenums="1"
script-option-defaults:
  plugin-depend:
    - 'Citizens'
```

Default: None (empty list)

### `file-logging-enabled`

Specifies if script file logging should be enabled. If this option is `true`, a script log file will be generated for the script, and any error messages (and print messages sent to the script's logger) will be logged to this file. If set to `false`, no messages will be logged to file, but messages will still be printed to the server console.

``` yaml linenums="1"
script-option-defaults:
  file-logging-enabled: true
```

Default: `true`

### `min-logging-level`

Specifies the minimum logging level that should be logged to the script's log file and the console for the script. Options can be found on the [JavaDocs](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Level.html).

``` yaml linenums="1"
script-option-defaults:
  min-logging-level: 'INFO'
```

Default: `INFO`

### `permission-default`

Specify a default value that permissions should have, if they do not have a `default` value defined. The allowed values for `permission-default` are `op`, `not op`, `true`, and `false`:

- `op`: Only server operators will have the permission node by default.
- `not op`: Players who are not operators will have the permission node by default.
- `true`: All players will have the permission (I.E. it is a default permission).
- `false`: No players will have the permission (I.E. it is *not* a default permission). 

``` yaml linenums="1"
script-option-defaults:
  permission-default: 'op'
```

Default: `op`

??? note "A note about default script permissions"

    The astute reader will notice that the `script-option-defaults` section of the config does not contain a `permissions` section to specify default script permissions. Originally, this feature existed, but it had to be removed, due to a quirk/bug in the way Bukkit parses permissions from the config file.
    
    In order to get this feature working for the `script_options.yml` file, an entirely custom config system had to be written for the script options config to parse permissions. I considered doing the same for the plugin config in order to allow the user to specify default permissions, however, I decided against this as I realized that specifying a default list of script permissions wouldn't really be useful to most users.


## `debug-options`

The configuration values in the `default-options` section are more advanced options that should only be changed if you encounter issues or would like to disable recommended features.

### `print-stack-traces`

Specifies if full stack traces should always be printed to the server console. Normally, if a Java exception occurs, a condensed version of the Java exception along with a Python traceback are printed, but if this parameter is set to `true`, a full Java stack trace will be printed in addition to a Python traceback.

``` yaml linenums="1"
debug-options:
  print-stack-traces: false
```

Default: `false`

### `show-update-messages`

Specifies whether the plugin should show messages in console and on join (to players with the permission `pyspigot.admin`) when a newer version of PySpigot is available to download on spigotmc.org. If set to `false`, no update messages will be shown, even if a newer version of the plugin is available.

``` yaml linenums="1"
debug-options:
  show-update-messages: true
```

Default: `true`

## Example Configuration

The following example configuration contains default values for all parameters.

``` yaml linenums="1"
# If false, will disable collection of metrics information by bStats for PySpigot. You may also disable bStats server-wide in the bStats config.yml under /plugins/bStats.
metrics-enabled: true
# The delay for loading scripts (in ticks) after the server finishes loading.
script-load-delay: 20
# List of relocation rules for libraries in the libs folder. Format as <pattern>|<relocated pattern>
library-relocations: []
# Date/time format for timestamps in script log files, written in Java's SimpleDateFormat pattern: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html
log-timestamp-format: 'MMM dd yyyy HH:mm:ss'
# If true, will print log messages to console every time a script is loaded, run, and unloaded.
script-action-logging: true
# If true, will log all redis events to the console and to a script's logger. If false, will only log reconnect events (reconnect attempts and failures)
verbose-redis-logging: true
# If true, scripts will be automatically unloaded if a plugin the script depends on is unloaded. This is especially useful to ensure script shutdown tasks that require a depending plugin complete successfully (prior to the plugin being unloaded).
script-unload-on-plugin-disable: true
# Options that pertain to Jython. Changing options in this section requires a server restart.
jython-options:
  # If true, the Jython runtime will be initialized during plugin load/server start. If false, the Jython runtime will not be initialized until the first script is loaded.
  init-on-startup: true
  # A list of system properties that will be passed to Jython. For a complete list, see https://javadoc.io/doc/org.python/jython-standalone/latest/org/python/core/RegistryKey.html
  properties:
    - 'python.cachedir.skip=true'
  # A list of args to pass to Jython when initialized. Equivalent to sys.argv in Python.
  args:
    - ''
# Default values for script options. If one or more options are not defined in the script_options.yml for the script, then PySpigot will fall back to these values.
script-option-defaults:
  # Whether the script is enabled
  enabled: true
  # An integer load priority for the script
  load-priority: 1
  # A list of plugins the script depends on
  plugin-depend: []
  # Whether script log messages should be logged to its respective log file
  file-logging-enabled: true
  # The minimum level to log to the console and to the script's log file
  min-logging-level: 'INFO'
  # The default permission level for permissions
  permission-default: 'op'
# Advanced debug options for scripts
debug-options:
  # If true, will print stack traces for all script-related exceptions to the server console
  print-stack-traces: false
  # If true, the plugin will show messages in console and on join (to players with the permission pyspigot.admin) when a newer version of PySpigot is available to download on spigotmc.org.
  show-update-messages: true
```