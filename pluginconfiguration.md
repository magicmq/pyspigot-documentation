# PySpigot Configuration

Here is PySpigot's configuration file. Comments should sufficiently explain each value:

```yaml
#If false, will disable collection of metrics information by bStats for PySpigot. You may also disable bStats server-wide in the bStats config.yml under /plugins/bStats.
metrics-enabled: true
# The delay for loading scripts (in ticks) after the server finishes loading.
script-load-delay: 20
#List of relocation rules for libraries in the libs folder. Format as <pattern>|<relocated pattern>
library-relocations: []
#If true, PySpigot will log script messages to their respective log file
log-to-file: true
#The minimum log level that will be logged to a script's log file. For log level and their order, see https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Level.html
min-log-level: 'INFO'
#Date/time format for timestamps in script log files, written in Java's SimpleDateFormat pattern: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html
log-timestamp-format: 'MMM dd yyyy HH:mm:ss'
# If true, will print log messages to console every time a script is loaded, run, and unloaded.
script-action-logging: true
#Advanced debug options for scripts
debug-options:
  #If true, will print stack traces for all script-related exceptions to the server console
  print-stack-traces: false
  # If ture, will suppress notifications that the plugin version is outdated
  suppress-update-messages: false
messages:
  plugin-prefix: '&8[&6PySpigot&8] &r'
```