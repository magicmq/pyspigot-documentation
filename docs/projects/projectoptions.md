# Project Options

PySpigot allows you to specify project-specific options by placing a file titled `project.yml` into the project's root folder.

Aside from the `main` option, the rest of the options outlined here are the same as regular [script options](../scripts/scriptoptions.md).

???+ tip

    Defining project options for each project is *optional*. See the [Defaults](#defaults) section for more information.

## Basic Format

If you would like to specify options for a project, create a file named `project.yml`, and place it into the *root directory* of the project folder. Then, add something like this to the file:

``` yaml linenums="1"
main: 'main.py'
enabled: true
file-logging-enabled: true
```

???+ notice

    The `project.yml` file *must* be named `project.yml`; no other names are accepted. Additionally, PySpigot searches *only* the root directory of the project for this file, so it must be placed there.

## Defaults

Specifying project options for each project is not a requirement. PySpigot relies on fallbacks as well as default values for project options in the event that they aren't defined in the project's `project.yml` file. The following diagram illustrates the process by which PySpigot searches for project options when a project loads.

``` mermaid
graph LR
  A[Project loads] --> B{Option present in project's project.yml?};
  B -->|Yes| C[Use option from project's project.yml]
  B -->|No| D{Default option present in config.yml?}
  D -->|Yes| E[Use default option defined in config.yml]
  D -->|No| F[Use internal default option]
```

PySpigot first checks if an option is present in the project's `project.yml` file. If the option isn't defined there, then it will fall back to whatever default value is defined in the [`config.yml` under `script-option-defaults`](../pyspigot/pluginconfiguration.md/#script-option-defaults). If a default option isn't specified here, then it will fall back to an internally defined default value.

PySpigot performs this process for **each project option** individually.

## Options

### `main`

Specify the main module for the project. PySpigot uses this to determine which module should be executed when the project loads. This is akin to calling `python main.py` for a multi-module Python project, where `main.py` is the main module for the project.

``` yaml linenums="1"
main: 'main.py'
```

**Default:** `main.py`

### `enabled`

Specify whether a project is enabled or disabled. To disable a project, set this value to `false`.

``` yaml linenums="1"
enabled: true
```

**Default:** `true`

### `load-priority`

Specify an integer load priority for the project. Scripts and projects are loaded in order from highest to lowest load priority. In other words, scripts/projects that have a higher load priority are loaded earlier, and scripts/projects with a lower load priority are loaded later. If multiple scripts and projects have the same load priority, they are loaded in alphabetical order.

``` yaml linenums="1"
load-priority: 1
```

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

**Default:** 1

### `plugin-depend`

Specify a list of plugins that this project requires to load. The project will not load if any of the plugin dependencies are not loaded and running on the server. Additionally, when a plugin is unloaded/disabled, any projects that depend on that plugin as specified under this option are automatically unloaded (if the `script-unload-on-plugin-disable` option in the `config.yml` is set to `true`).

``` yaml linenums="1"
plugin-depend: ['Citizens', 'Vault']
```

???+ notice

    If you are working with ProtocolLib or PlaceholderAPI in your project, you **do not** need to specify either of them here. PySpigot has built-in support for these two plugins, and the dependency management is handled internally.

**Default:** None (empty list)

### `file-logging-enabled`

Specify if project file logging should be enabled for the project. If this option is `true`, a project log file will be generated, and any error messages (and print messages sent to the project's logger) will be logged to this file. If this option is `false`, no messages will be logged to a log file, but messages will still be printed to the server console.

``` yaml linenums="1"
file-logging-enabled: true
```

**Default:** `true`

### `min-logging-level`

Specify the minimum logging level that should be logged to the project's log file and the console. Options can be found on the [JavaDocs](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Level.html).

``` yaml linenums="1"
min-log-level: 'INFO'
```

**Default:** `INFO`

### `permissions`

Specify a list of permissions that the project uses. This is useful for projects that want to restrict access to certain features. This section is defined in the [exact same way](https://docs.papermc.io/paper/dev/plugin-yml#permissions) that permissions are defined in the `plugin.yml` file for a Bukkit plugin. See usage code example below for how to define permissions, defaults, and child permissions.

``` yaml linenums="1"
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