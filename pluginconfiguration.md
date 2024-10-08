# PySpigot Configuration

Here is PySpigot's configuration file. Comments should sufficiently explain each value:

```yaml
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
# Default values for script options. If one or more options are not defined in the script_options.yml for the script, then PySpigot will fall back to these values.
script-option-defaults:
  # Whether the script is enabled
  enabled: true
  # A list of other scripts the script depends on
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
  # If false, the pyspigot.py file will *not* be automatically replaced with the most recent version (from the pyspigot JAR) on server start if a change is detected. It is recommended to keep this enabled in the event a plugin update comes with an updated pyspigot.py file.
  auto-pyspigot-lib-update-enabled: true
messages:
  plugin-prefix: '&8[&6PySpigot&8] &r'
```