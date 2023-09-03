# PySpigot

> A Python scripting engine for your Minecraft server.

## What is PySpigot?

PySpigot is a Python scripting engine that allows you to write Python scripts that can interface with Bukkit and other parts of a Minecraft server seamlessly, without writing a plugin. Scripts have full access to the Java class path at runtime (meaning anything is possible!) and also have access to PySpigot's managers, which help make writing scripts much easier.

## Features

- Load scripts on server start and via commands
- Stop, reload, and unload server scripts via commands
- Register event listeners
- Register commands
- Schedule tasks (synchronous and asynchronous)
- Work with config files
- Register ProtocolLib packet listeners
- Register PlaceholderAPI placeholder expansions
- Comprehensive logging of errors and exceptions on a per-script basis, to file
- Load Java libraries you'd like to work with at runtime
- Write scripts in Python syntax
- Scripts have complete access to the Bukkit/Spigot API, as well as APIs of other plugins, so anything is possible.
- And more!

## Getting Started

See the [Quick Start Guide](quickstart.md) for a brief tutorial. Check out the rest of the documentation for more advanced usage.

## Examples

Check out any of the [examples](examples.md) to see some examples of PySpigot scripts.

## Updates

When PySpigot is updated to a newer version, this update will be pushed to Spigot, as well as the official GitHub repository.

If you're running an outdated version of PySpigot, you'll see notifications in two places as a reminder to update:

- In the console, when the plugin loads (such as on server start).
- In chat when you log into your server. This message is only displayed to players with the permission `pyspigot.admin`.

?> It is recommended that you always use the latest version of PySpigot. Because the plugin is in its early stages of developmnt, newer versions likely contain key bug fixes and improvements.

## Links

- PySpigot's GitHub repository can be viewed [here](https://github.com/magicmq/pyspigot).
- Also check out Jython on Github [here](https://github.com/jython/jython).
- JavaDocs for the project can be viewed [here](https://javadocs.magicmq.dev/pyspigot/).

### Discord

Join the official PySpigot Discord server here: https://discord.gg/f2u7nzRwuk

## Metrics

PySpigot uses [bStats](https://bstats.org/) to collect anonymous usage data for PySpigot. If you would like to disable this feature, set `metrics-enabled` to `false` in PySpigot's config.yml. Alternatively, you can disable bStats server-wide by setting `enabled` to `false` in /plugins/bStats/config.yml.

## Licensing

Please check out the [License Info](#license.md) page for information on licensing.