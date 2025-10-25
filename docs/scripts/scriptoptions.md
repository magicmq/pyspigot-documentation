# Script Options

In PySpigot's main plugin folder, you'll find a file titled `script_options.yml`. This file enables you to specify various script-specific options, all of which are covered here.

???+ tip

    Defining script options for each script is *optional*. See the [Defaults](#defaults) section for more information.

## Basic Format

If you would like to specify options for a script, do it like so:

``` yaml linenums="1"
test.py: # (1)!
  enabled: true
  file-logging-enabled: true
```

1.  All options defined under this section apply to the script `test.py` only.

Each section in the `script_options.yml` file should be the name of a script. The name must *exactly* match the script's name, (including `.py`), or the options won't be parsed for that script.

All options within the section apply to the script by which the section is named. In the above example, the options `enabled` and `file-logging-enabled` apply to the script `test.py`.

### Defining Options for Multiple Scripts

If you would like to define options for multiple scripts, do it like so:

``` yaml linenums="1"
test.py: # (1)!
  enabled: true
  file-logging-enabled: true
test2.py: # (2)!
  enabled: true
  load-priority: 10
```

1.  The options defined under this section apply to the script `test.py`.
2.  The options defined under this section apply to the script `test2.py`.

As you can see, in the above example, we define a new section titled `test2.py`, and all options within this section will apply to the `test2.py` script. All options within the `test.py` script still apply only to the script `test.py`.

## Defaults

Specifying script options for each script is not a requirement. PySpigot relies on fallbacks as well as default values for script options in the event that they aren't defined in the `script_options.yml` file for the script that is being loaded. The following diagram illustrates the process by which PySpigot searches for script options when a script loads.

``` mermaid
graph LR
  A[Script loads] --> B{Option present in script_options.yml?};
  B -->|Yes| C[Use option from script_options.yml]
  B -->|No| D{Default option present in config.yml?}
  D -->|Yes| E[Use default option defined in config.yml]
  D -->|No| F[Use internal default option]
```

PySpigot first checks if an option is present in the `script_options.yml` file. If the option isn't defined here, then it will fall back to whatever default value is defined in the [`config.yml` under `script-option-defaults`](../pyspigot/pluginconfiguration.md/#script-option-defaults). If a default option isn't specified here, then it will fall back to an internally defined default value.

PySpigot performs this process for **each script option** individually.

## Options

### `enabled`

Specify whether a script is enabled or disabled. To disable a script, set this value to `false`.

``` yaml linenums="1"
test.py:
  enabled: true
```

**Default:** `true`

### `load-priority`

Specify an integer load priority for the script. Scripts and projects are loaded in order from highest to lowest load priority. In other words, scripts/projects that have a higher load priority are loaded earlier, and scripts/projects with a lower load priority are loaded later. If multiple scripts and projects have the same load priority, they are loaded in alphabetical order.

``` yaml linenums="1"
test.py:
  load-priority: 1
```

**Default:** 1

### `plugin-depend`

Specify a list of plugins that this script requires to load. The script will not load if any of the plugin dependencies are not loaded and running on the server. Additionally, when a plugin is unloaded/disabled, any scripts that depend on that plugin as specified under this option are automatically unloaded (if the `script-unload-on-plugin-disable` option in the `config.yml` is set to `true`).

``` yaml linenums="1"
test.py:
  plugin-depend: ['Citizens', 'Vault']
```

???+ notice

    If you are working with ProtocolLib or PlaceholderAPI in your script, you **do not** need to specify either of them here. PySpigot has built-in support for these two plugins, and the dependency management is handled internally.

**Default:** None (empty list)

### `file-logging-enabled`

Specify if script file logging should be enabled for the script. If this option is `true`, a script log file will be generated, and any error messages (and print messages sent to the script's logger) will be logged to this file. If this option is `false`, no messages will be logged to a log file, but messages will still be printed to the server console.

``` yaml linenums="1"
test.py:
  file-logging-enabled: true
```

**Default:** `true`

### `min-logging-level`

Specify the minimum logging level that should be logged to the script's log file and the console. Options can be found on the [JavaDocs](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Level.html).

``` yaml linenums="1"
test.py:
  min-log-level: 'INFO'
```

**Default:** `INFO`

### `permissions`

???+ warning

    Script permissions are **not** supported on Velocity and BungeeCord. This section of the script options configuration has **no effect** on those platforms.

Specify a list of permissions that the script uses. This is useful for scripts that want to restrict access to certain features. This section is defined in the [exact same way](https://docs.papermc.io/paper/dev/plugin-yml#permissions) that permissions are defined in the `plugin.yml` file for a Bukkit plugin. See usage code example below for how to define permissions, defaults, and child permissions.

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

**Default:** None (no permissions defined)

### `permission-default`

???+ warning

    Script are **not** supported on Velocity and BungeeCord. This section of the script options configuration has **no effect** on those platforms.

Specify a default value that permissions should have, if they do not have a `default` value defined.

``` yaml linenums="1"
test.py:
  permission-default: true
```

The allowed values for `permission-default` are `op`, `not op`, `true`, and `false`:

- `op`: Only server operators will have the permission node by default.
- `not op`: Players who are not operators will have the permission node by default.
- `true`: All players will have the permission (I.E. it is a default permission).
- `false`: No players will have the permission (I.E. it is *not* a default permission). 

**Default:** `op`