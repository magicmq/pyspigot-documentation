---
title: Home
---

<div style="text-align: center;" markdown>
# **PySpigot**
</div>

> A Python scripting engine for your Minecraft server.

<div style="text-align: center;" markdown>
[![GitHub release (latest by date)](https://img.shields.io/github/v/release/magicmq/pyspigot?label=Release&style=plastic)](https://github.com/magicmq/pyspigot/releases/latest)
[![Latest snapshot](https://img.shields.io/badge/dynamic/xml?label=Latest%20snapshot&style=plastic&query=%2F%2Fmetadata%2Fversioning%2Fversions%2Fversion%5Blast()%5D&url=https%3A%2F%2Frepo.magicmq.dev%2Frepository%2Fmaven-snapshots%2Fdev%2Fmagicmq%2Fpyspigot%2Fmaven-metadata.xml)](https://ci.magicmq.dev/job/PySpigot/)
[![Develop GitHub commits since latest stable release (by SemVer)](https://img.shields.io/github/commits-since/magicmq/pyspigot/latest/master?label=Commits%20since%20last%20release&style=plastic)](https://github.com/magicmq/pyspigot)
</div>

## What is PySpigot?

PySpigot is a Spigot/Bukkit plugin that acts as a Python scripting engine that runs in Minecraft. Users can write Python scripts, which, for lack of a better term, act as "mini-plugins". Because PySpigot runs entirely in the Minecraft environment on Java, scripts have access to the full server, including the entire Bukkit API, as well as the APIs of other plugins. With PySpigot, you can write a script that does the exact same thing a plugin can but in a small fraction of the time. PySpigot is also excellent for individuals who know Python, but not Java. PySpigot is a lot like [Skript](https://docs.skriptlang.org/), but much more powerful.

## What Can PySpigot Do?

<div class="grid cards" markdown>

-   :material-clock-fast:{ .lg .middle } __Write Scripts Quickly, in Python__

    ---

    PySpigot scripts can do all the same things that a plugin can do. However, PySpigot scripts can be written in a fraction of the time it takes to write a full plugin.

    PySpigot is also great for people who know Python, but not Java - no Java experience is required!

-   :octicons-project-roadmap-24:{ .lg .middle } __Write Multi-Module Projects__

    ---

    PySpigot fully supports writing multi-module projects. These function similarly to multi-module Python projects and offer more ideal code organization and structuring.

-   :octicons-smiley-24:{ .lg .middle } __Easy to Use__

    ---

    PySpigot's features are designed with ease of use in mind. It is easy to use for simple tasks, but you can also create really complex scripts. The possibilities are endless!

-   :octicons-cpu-24:{ .lg .middle } __Unrestricted Bukkit API Access__

    ---

    Because PySpigot runs inside of the Minecraft/Java environment, PySpigot scripts have complete access to the Bukkit/Spigot API.
    
    Register event listeners, create commands, schedule tasks, work with config files, and more!

-   :octicons-bug-24:{ .lg .middle } __Comprehensive Error Logging__

    ---

    PySpigot includes comprehensive error logging on a per-script basis, both to console and to a script-specific log file, making debugging as easy as possible.

-   :octicons-package-24:{ .lg .middle } __Full Support for External Java/Python Libraries__

    ---

    PySpigot includes full support for loading and working with external libraries such as the [Apache Commons](https://commons.apache.org/) libraries.

-   :octicons-pencil-24:{ .lg .middle } __Highly Configurable__

    ---

    PySpigot was designed to be as configurable as possible. There are several editable script-specific options, as well as a variety of plugin-wide options that can be configured.


-   :octicons-database-24:{ .lg .middle } __Full MySQL, MongoDB, and Redis Support__

    ---

    PySpigot includes support for MySQL, MongoDB, and Redis and greatly simplifies these 

-   :octicons-code-24:{ .lg .middle } __Full ProtocolLib Support__

    ---

    PySpigot has built-in support for ProtocolLib, making it easy to work with packets. There is full support for registering packet listeners, packet modification, and sending packets.

-   :octicons-code-24:{ .lg .middle } __Full PlaceholderAPI Support__

    ---

    PySpigot also has built-in support for PlaceholderAPI, making it easy to register placeholder expansions.

-   :material-scale-balance:{ .lg .middle } __Open Source, Apache 2.0 License__

    ---

    PySpigot's source is fully available on [GitHub](https://github.com/magicmq/PySpigot), and it's licensed under the [Apache 2.0 license](misc/license.md).

-   :octicons-star-fill-24:{ .lg .middle } __And More!__

    ---

    I am constantly working on new features and improvements to make scripting in Minecraft the best possible experience.

    Ideas are always welcome. If you have an idea for a new feature, [submit a feature request](https://github.com/magicmq/pyspigot/issues).

</div>

## Examples

Check out any of the [examples](scripts/examples.md) to see some examples of PySpigot scripts.

## Getting Started

See the [Quick Start Guide](pyspigot/quickstart.md) for a brief tutorial. Check out the rest of the documentation for more advanced usage.

## Discord

PySpigot has an official Discord server with help channels, bug reporting, and more. I'm usually active on Discord, so if you want to reach me, joining the Discord server is an excellent way to do so. [Join the Discord server here.](https://discord.gg/f2u7nzRwuk)

## Metrics

PySpigot uses [bStats](https://bstats.org/) to collect anonymous usage data for PySpigot. I use these data to inform me about PySpigot's users, including which country they are from (so that I can offer support in popular non-English languages) as well as what Minecraft and Java versions are most popular with users. bStats also collects some other useful data, including server software (Spigot, Paper, Purpur, etc.), plugin version, and number of scripts loaded. Sensitive or identifying information is not collected.

If you would like to opt out of this feature, set `metrics-enabled` to `false` in PySpigot's config file. Alternatively, you can disable bStats server-wide by setting `enabled` to `false` in /plugins/bStats/config.yml.

## Licensing Information

PySpigot is licensed using the Apache 2.0 license. Please check out the [License Info](misc/license.md) page for more information.