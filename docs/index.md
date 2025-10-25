---
title: Home
---

<div style="text-align: center;" markdown>
# **PySpigot**
</div>

> A Python scripting engine for Minecraft.

<div style="text-align: center;" markdown>
[![GitHub release (latest by date)](https://img.shields.io/github/v/release/magicmq/pyspigot?label=Release&style=plastic)](https://github.com/magicmq/pyspigot/releases/latest)
[![Latest snapshot](https://img.shields.io/badge/dynamic/xml?label=Latest%20snapshot&style=plastic&query=%2F%2Fmetadata%2Fversioning%2Fversions%2Fversion%5Blast()%5D&url=https%3A%2F%2Frepo.magicmq.dev%2Frepository%2Fmaven-snapshots%2Fdev%2Fmagicmq%2Fpyspigot%2Fmaven-metadata.xml)](https://ci.magicmq.dev/job/PySpigot/)
[![Develop GitHub commits since latest stable release (by SemVer)](https://img.shields.io/github/commits-since/magicmq/pyspigot/latest/master?label=Commits%20since%20last%20release&style=plastic)](https://github.com/magicmq/pyspigot)
</div>

## What is PySpigot?

PySpigot is a modular plugin system that brings Python scripting to both Minecraft servers (running Spigot/Paper or their forks), and proxy platorms including BungeeCord and Velocity. It acts as a Python scripting engine that runs directly within the Minecraft environment on Java, allowing users to write Python scripts that function as lightweight "mini-plugins." These scripts have access to the full capabilities of the host platform, whether it’s the Bukkit API on Spigot, or the Bungee and Velocity APIs on their respective proxies, along with APIs from other installed plugins. With PySpigot, you can create scripts that could accomplish anything a traditional plugin can, but in a fraction of the overhead and development time. It's an ideal solution for developers who are familiar with Python but not Java. In essence, PySpigot is similar to [Skript](https://docs.skriptlang.org/), but much more powerful and flexible.

## What Can PySpigot Do?

<div class="grid cards" markdown>

-   :material-clock-fast:{ .lg .middle } __Write Scripts Quickly, in Python__

    ---

    PySpigot scripts can do everything a traditional plugin can — across Spigot/Bukkit servers and proxy platforms like BungeeCord and Velocity — but can be written in a fraction of the time.

    PySpigot is also great for people who know Python, but not Java — no Java experience is required!

-   :octicons-project-roadmap-24:{ .lg .middle } __Write Multi-Module Projects__

    ---

    PySpigot fully supports writing multi-module projects, allowing you to organize your code across multiple scripts or shared modules. This mirrors the structure of larger Python projects and enables clean, maintainable design.

-   :octicons-smiley-24:{ .lg .middle } __Easy to Use__

    ---

    PySpigot is designed with usability in mind. It’s simple enough for quick automation tasks, yet powerful enough for complex plugin-level systems — whether you’re scripting on a single server or managing a large proxy network.

-   :octicons-cpu-24:{ .lg .middle } __Unrestricted API Access__

    ---

    Because PySpigot runs directly within the Java environment, scripts have complete access to the full API of the platform they run on — including the Bukkit API, or the BungeeCord and Velocity APIs.
    
    Register events, create commands, manage players, schedule tasks, and interact with other plugins — all from Python!

-   :octicons-bug-24:{ .lg .middle } __Comprehensive Error Logging__

    ---

    PySpigot includes detailed per-script error logging — both in the console and in individual log files — making it easy to debug issues and identify script-level problems.

-   :octicons-package-24:{ .lg .middle } __Full Support for External Java/Python Libraries__

    ---

    PySpigot includes full support for loading and using external Java libraries and Python modules.

-   :octicons-pencil-24:{ .lg .middle } __Highly Configurable__

    ---

    PySpigot was built with flexibility in mind. It includes script-specific and global configuration options that let you tailor runtime behavior, library management, and script isolation to your exact needs.


-   :octicons-database-24:{ .lg .middle } __Full MySQL, MongoDB, and Redis Support__

    ---

    PySpigot includes support for MySQL, MongoDB, and Redis, simplifying data storage and communication across both single servers and networked proxies.

-   :octicons-code-24:{ .lg .middle } __Full ProtocolLib Support__

    ---

    On Spigot/Bukkit servers, PySpigot integrates directly with ProtocolLib, allowing you to easily work with packets — register listeners, modify packets, or send custom data to clients.

-   :octicons-code-24:{ .lg .middle } __Full PlaceholderAPI Support__

    ---

    PySpigot provides seamless integration with PlaceholderAPI, making it simple to register placeholder expansions and interact with placeholder data in your scripts.

-   :material-scale-balance:{ .lg .middle } __Open Source, Apache 2.0 License__

    ---

    PySpigot's source is fully available on [GitHub](https://github.com/magicmq/PySpigot), and it's licensed under the [Apache 2.0 license](misc/license.md).

-   :octicons-star-fill-24:{ .lg .middle } __And More!__

    ---

    PySpigot is constantly evolving with new features and improvements to make scripting in Minecraft more powerful and enjoyable.

    Have an idea? [Submit a feature request](https://github.com/magicmq/pyspigot/issues) — community contributions and feedback are always welcome!

</div>

## Examples

Check out any of the [examples](scripts/examples.md) to see some examples of PySpigot scripts.

## Getting Started

See the [Quick Start Guide](pyspigot/quickstart.md) for a brief tutorial. Check out the rest of the documentation for more advanced usage.

## Discord

PySpigot has an official Discord server with help channels, bug reporting, and more. It's a growing community, with several active users. If you're looking for help or want to chat about PySpigot, [Join the Discord server here.](https://discord.gg/f2u7nzRwuk).

## Metrics

PySpigot uses [bStats](https://bstats.org/) to collect anonymous usage data for PySpigot. I use these data to inform me about PySpigot's usage, including which Minecraft versions and Java versions are most popular. bStats also collects some other useful data, including server software (Spigot, Paper, Purpur, BungeeCord, Velocity, etc.), plugin version, and number of scripts loaded. Sensitive or privileged information is not collected.

If you would like to opt out of this feature, set `metrics-enabled` to `false` in PySpigot's config file. Alternatively, you can disable bStats server-wide by setting `enabled` to `false` in `/plugins/bStats/config.yml`.

## Licensing Information

PySpigot is licensed using the Apache 2.0 license. Visit the [License Info](misc/license.md) page for more information.