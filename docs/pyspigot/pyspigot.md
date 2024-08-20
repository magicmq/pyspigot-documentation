# General Information about PySpigot

## How Does PySpigot Work?

A standard Python installation runs CPython. CPython is an implementation of Python that runs on the C programming language. CPython is perhaps the most popular Python implementation, since it is the reference implementation of python, or, in other words, the default implementation. Several other implementations of Python exist, including Cython, IronPython, Nuitka, Numba, PyPy, and Stackless Python. Jython is another Python implementation.

Just as CPython is an implementation of Python that runs on the C programming language, Jython is an implementation of Python that runs on Java. Because Minecraft also runs on Java, Jython can run within the same environment that Minecraft runs. Indeed, PySpigot ships with Jython bundled in. Jython supports interpretation of Python code on the fly, which is essentially how scripts are run.

The real magic of Jython is that, because it runs within the same environment that Minecraft does, all of the code on the Minecraft side of things is fully available to Jython, and by extension, to scripts that are run on Jython. This makes it possible to reference Java classes within the Python code. The integration is almost completely seamless. Jython also handles conversion of variables and data structures between Python and Java types automatically.

### One Noteable Drawback Regarding Jython

The major drawback of Jython is that it currently only supports Python 2. Work towards a Python 3 implementation is currently ongoing over at [Jython's GitHub repository](https://github.com/jython/jython).

Regarding different avenues, other Python-Java interop projects support Python 3. One such example is [Py4J](https://www.py4j.org/), which is more akin to a "network bridge" between the Python and Java runtimes rather than a true Python implementation. Although it supports Python 3, Py4J would be incredibly difficult to implement as a scripting engine for Bukkit, as it relies heavily on time-consuming I/O operations and Callbacks, which would make the Minecraft server quite unstable. Additionally, Py4J would require a CPython installation (regular Python) on the same machine, a rather difficult requirement to fulfill when working within a containerized Minecraft instance (such as Pterodactyl, shared hosting providers, etc.).

When I began this project, I looked at using several libraries, Py4J included. I came to the conclusion that, although Jython only supports Python 2, it has several key advantages over other libraries in this specific use case, including:

- It runs entirely on the JVM, making it very easy to interface with Python code from the Java side.
- It runs entirely on the JVM, which gives all Python code direct access to the entire Java classpath at runtime. Ergo, Python code has full access to Java code and vice versa.
- It does not require any external Python installation to work. Drag and drop PySpigot into your plugins folder, and you're good to go.
- And finally, it is reasonably fast (fast enough for Minecraft's standards), given that, again, it runs entirely on the JVM.

Thus, for the foreseeable future, PySpigot will continue to utilize Jython.

## Metrics

PySpigot uses [bStats](https://bstats.org/) to collect anonymous usage data for PySpigot. I use these data to inform me about PySpigot's users, including which country they are from (so that I can offer support in popular non-English languages) as well as what Minecraft and Java versions are most popular with users. bStats also collects some other useful data, including server software (Spigot, Paper, Purpur, etc.), plugin version, and number of scripts loaded. Sensitive or identifying information is not collected.

If you would like to opt out of this feature, set `metrics-enabled` to `false` in PySpigot's config file. Alternatively, you can disable bStats server-wide by setting `enabled` to `false` in /plugins/bStats/config.yml.

## Updates

When PySpigot is updated to a newer version, this update will be pushed to Spigot, as well as the official GitHub repository.

If you're running an outdated version of PySpigot, you'll see notifications in two places as a reminder to update:

- In the console, when the plugin loads (such as on server start).
- In chat when you log into your server. This message is only displayed to players with the permission `pyspigot.admin`.

!!! tip

    It is recommended that you always use the latest version of PySpigot. Because the plugin is in its early stages of development, newer versions will usually contain important bug fixes and improvements.