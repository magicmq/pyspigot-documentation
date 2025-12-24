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

### `main`

Specifies the main module for a project (this option is not applicable to a single-file script). PySpigot uses this to determine which module should be executed when the project loads. This is akin to calling `python main.py` for a multi-module Python project, where `main.py` is the main module for the project.

``` yaml linenums="1"
main: 'main.py'
```

Default: `main.py`

### `enabled`

Used to enable or disable a script or project. To disable a script/project, set this value to `false`.

``` yaml linenums="1"
script-option-defaults:
  enabled: true
```

Default: `true`

### `load-priority`

Specifies an integer load priority for the script or project. Scripts and projects are loaded in order from highest to lowest load priority. In other words, scripts/projects that have a higher load priority are loaded earlier, and scripts/projects with a lower load priority are loaded later. If multiple scripts/projects have the same load priority, they are loaded in alphabetical order.

``` yaml linenums="1"
script-option-defaults:
  load-priority: 1
```

Default: `1`

#### Load Priority Example

Because load priority is somewhat difficult to understand conceptually, here is an example of the behavior:

| Script/Project Name | Load Priority |
| ------------------- | ------------- |
| script.py           | 1             |
| test.py             | 5             |
| util.py             | 99            |
| economy.py          | 100           |
| player.py           | Not set       |
| Punisher (project)  | Not set       |
| Chat (project)      | 5             |
| other.py            | 10            |

PySpigot would load the above scripts/projects in the following order:

1. economy.py *(Loads first)*
2. util.py
3. other.py
4. Chat
5. test.py
6. player.py
7. Punisher
8. script.py *(Loads last)*

### `plugin-depend`

Specifies a list of plugins that the script or project depends on. The script/project will not load if any of the plugin dependencies are not loaded and running on the server. Additionally, if the [#script-unload-on-plugin-disable] parameter is set to `true`, any scripts/projects that depend on that plugin as specified under this option are automatically unloaded when the plugin is unloaded/disabled.

??? note

    If you are working with ProtocolLib or PlaceholderAPI in your script or project, you **do not** need to specify either of them here. PySpigot has built-in support for these two plugins, and the dependency management is handled automatically.

``` yaml linenums="1"
script-option-defaults:
  plugin-depend:
    - 'Citizens'
```

Default: None (empty list)

### `file-logging-enabled`

Specifies if script/project file logging should be enabled. If this option is `true`, a script/project log file will be generated, and any error messages (and print messages sent to the script/project's logger) will be logged to this file. If set to `false`, no messages will be logged to a log file, but messages will still be printed to the server console.

``` yaml linenums="1"
script-option-defaults:
  file-logging-enabled: true
```

Default: `true`

### `min-logging-level`

Specifies the minimum logging level that should be logged to the script/project's log file and the console. Options can be found on the [JavaDocs](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Level.html).

``` yaml linenums="1"
script-option-defaults:
  min-logging-level: 'INFO'
```

Default: `INFO`

### `permissions`

Specify a list of permissions that the script/project uses. This is useful for scripts or projects that want to restrict access to certain features. This section is defined in the [exact same way](https://docs.papermc.io/paper/dev/plugin-yml#permissions) that permissions are defined in the `plugin.yml` file for a Bukkit plugin. See usage code example below for how to define permissions, defaults, and child permissions.

``` yaml linenums="1"
test.py:
  permissions:
    permission.node.*:
      description: 'This is a permission node'
      default: op
      children:
        permission.node.child: true
    permission.node.child:
      description: 'This is a child permission node'
      default: true
    another.permission.node:
      description: 'This is another permission node'
      default: not op
```

- `description` is a description of the permission node, and this is what will be displayed in the permissions list. The default value is the name of the permission node.
- `default` is the default value of the permission node, or, in other words, who should have the permission node by default. There are four possible values for `default`: 
  - `op`: Only server operators will have the permission node by default.
  - `not op`: Players who are not operators will have the permission node by default.
  - `true`: All players will have the permission (I.E. it is a default permission).
  - `false`: No players will have the permission (I.E. it is *not* a default permission). The default value is the value of `default_permission` (outlined below).
- `children` is a list of child permissions that should inherit from the parent permission. Each permission node may have children. When set to `true`, the child will inherit the parent permission.

Default: None (no permissions defined)

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

### `jython-logging-level`

Specifies the logging level at which the internal Jython logger logs messages. It can be useful to set this to `FINE` or `ALL` for debugging purposes, to get a better sense of what Jython is doing internally. Note that for Jython log messages to show at the appropriate level, the server-wide logging level would also need to be adjusted accordingly.

``` yaml linenums="1"
debug-options:
  jython-logging-level: 'INFO'
```

Default: `INFO`

### `patch-threading`

Specifies whether or not a patch should be applied to the `threading` module on script unload. This patch is aimed at fixing a bug that occurs when using the threading module from within an asynchronous task. For more information, see [this GitHub issue report](https://github.com/magicmq/pyspigot/issues/18#issue-3012022678). Under most circumstances, this should be set to `true`.

``` yaml linenums="1"
debug-options:
  patch-threading: true
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
  # For projects, the main script file for the project.
  main: 'main.py'
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
  # The logging level for Jython internals. Can be useful to set this to FINE or ALL for debugging purposes. Note: the server's root logger will also need to be configured to accept debug messages for Jython's debug messages to show.
  jython-logging-level: 'INFO'
  # If true, PySpigot will patch the threading module on script unload (if it's being used in the script) in order to prevent the server from hanging. For more information, see https://github.com/magicmq/pyspigot/issues/18#issue-3012022678
  patch-threading: true
```