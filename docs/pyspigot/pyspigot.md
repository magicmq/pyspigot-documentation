# General Information about PySpigot

## How Does PySpigot Work?

A standard Python installation runs CPython. CPython is an implementation of Python that runs on the C programming language. CPython is perhaps the most popular Python implementation, since it is the reference implementation of python, or, in other words, the default implementation. Several other implementations of Python exist, including Cython, IronPython, Nuitka, Numba, PyPy, and Stackless Python. PySpigot utilizes [Jython](https://www.jython.org/), another implementation of Python.

Just as CPython is an implementation of Python that runs on the C programming language, Jython is an implementation of Python that runs on Java. Because Minecraft also runs on Java, Jython can run within the same environment that Minecraft runs. Indeed, PySpigot ships with Jython bundled in, and it runs on top of the Minecraft server instance. Additionally, Jython supports interpretation of Python code on the fly, which is essentially how scripts are run.

The real magic of Jython is that, because it runs within the same environment that Minecraft does, all of the code on the Minecraft side of things is fully available to Jython, and by extension, to scripts that are executed with Jython. This makes it possible to reference Java classes within the Python code. The integration is almost completely seamless. Jython also handles conversion of variables and data structures between Python and Java types automatically.

### One Noteable Drawback Regarding Jython

The major drawback of Jython is that it currently only supports Python 2. Work towards a Python 3 implementation is currently ongoing over at [Jython's GitHub repository](https://github.com/jython/jython).

Regarding different avenues, other Python-Java interop projects support Python 3. One such example is [Py4J](https://www.py4j.org/), which is more akin to a "network bridge" between the Python and Java runtimes rather than a true Python implementation. Although it supports Python 3, Py4J would be incredibly difficult to implement as a scripting engine for Bukkit, as it relies heavily on time-consuming I/O operations and Callbacks, which would make the Minecraft server quite unstable. Additionally, Py4J would require a CPython installation (regular Python) on the same machine, a rather difficult requirement to fulfill when working within a containerized Minecraft instance (such as Pterodactyl, shared hosting providers, etc.).

When I began this project, I looked at using several libraries, Py4J included. I came to the conclusion that, although Jython only supports Python 2, it has several key advantages over other libraries in this specific use case, including:

- It runs entirely on the JVM, making it very easy to interface with Python code from the Java side.
- It runs entirely on the JVM, which gives all Python code direct access to the entire Java classpath at runtime. Ergo, Python code has full access to Java code and vice versa.
- It does not require any external Python installation to work. Drag and drop PySpigot into your plugins folder, and you're good to go.
- And finally, it is reasonably fast (fast enough for Minecraft's standards), given that, again, it runs entirely on the JVM.

Thus, for the foreseeable future, PySpigot will continue to utilize Jython.

## The PySpigot Plugin Folder

The PySpigot plugin folder is the central repository where scripts, config files, libraries, and data are stored. It has the following structure:

``` py
plugins/
├─ PySpigot/ # (1)!
│  ├─ configs/ # (2)!
|  |  ├─ test.yml
|  |  └─ ...
│  ├─ java-libs/ # (3)!
|  |  ├─ lib-example.jar
|  |  └─ ...
|  ├─ logs/ # (4)!
|  |  ├─ test.log
|  |  └─ ...
|  ├─ python-libs/ # (5)!
|  |  ├─ lib-example.py
|  |  └─ ...
|  ├─ scripts/ # (6)!
|  |  ├─ test.py
|  |  └─ ...
|  ├─ config.yml # (7)!
|  └─ script_options.yml # (8)!
└─ ...
```

1.  The main plugin folder.
2.  The `configs` folder is where script config files and other data files (json, etc.) are stored, by default. The `configs` folder may contain subfolders for more optimal organization of a script's config and other data files.
3.  The `java-libs` folder is where external Java libraries should be placed. All JAR files in this folder are automatically loaded when the plugin loads.
4.  The `logs` folder is where script log files are stored. A script's log file will have the same name as the script (except it will end in `.log` instead of `.py`).
5.  The `python-libs` folder is where external Python modules should be placed. Any external Python modules (that end in `.py`) will be automatically accessible (able to be imported) from scripts.
6.  The `scripts` folder is where all PySpigot scripts live. The `scripts` folder may contain subfolders for more optimal organization of scripts.
7.  This is the main `config.yml` file for PySpigot. For more detailed information on the config, see the [configuration page](pluginconfiguration.md) on the documentation.
8.  This is the `script_options.yml` file. This is where script options for all scripts should be placed. For more detailed information on script options, see the [script options page](../scripts/scriptoptions.md) on the documentation.

## Metrics

PySpigot uses [bStats](https://bstats.org/) to collect anonymous usage data for PySpigot. I use these data to inform me about PySpigot's users, including which country they are from (so that I can offer support in popular non-English languages) as well as what Minecraft and Java versions are most popular with users. bStats also collects some other useful data, including server software (Spigot, Paper, Purpur, etc.), plugin version, and number of scripts loaded. Sensitive or identifying information is not collected.

If you would like to opt out of this feature, set `metrics-enabled` to `false` in PySpigot's config file. Alternatively, you can disable bStats server-wide by setting `enabled` to `false` in /plugins/bStats/config.yml.

## Updates

When PySpigot is updated to a newer version, this update will be pushed to Spigot, as well as the official GitHub repository. PySpigot has an automated system that checks Spigot for any available plugin updates.  If PySpigot finds a more up to date version available on Spigot, it will send notifications in two places:

- In the console, when the plugin loads (such as on server start).
- In chat when you log into your server. This message is only displayed to players with the permission `pyspigot.admin`.

To disable these update messages, set `show-update-messages` to `false` in the config.yml.

???+ tip

    It is recommended that you always use the latest version of PySpigot. Because the plugin is in its early stages of development, newer versions will usually contain important bug fixes and improvements.

## Security and Permissions

???+ quote-custom "Inspirational Quote"

    ***"With great power comes great responsibility."** - Uncle Ben to Peter Parker, Spider-Man*

PySpigot is an incredibly powerful plugin, and with such power, much damage could be done. Unlike other plugins, PySpigot is virtually unlimited in what it can access and in what it can do on the server. Therefore, you are advised to use **extreme** caution when deciding who to grant permissions to. This includes permissions to write scripts, load scripts, and unload scripts. See the next section for a list of permissions PySpigot defines as well as a description of each.

PySpigot does not include any security safeguards outside of required permissions to use commands, so judicious usage of the plugin is advised.

### Permissions

| Permission                      | Description                                                                                           |
| ------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `pyspigot.admin`                | If a plugin update is available, users with this permission will be shown an update message on login. |
| `pyspigot.command.listcmds`     | Grants permission to list all possible subcommands of `/pyspigot`.                                    |
| `pyspigot.command.help`         | Grants permission to use the `/pyspigot help` command.                                                |
| `pyspigot.command.info`         | Grants permission to use the `/pyspigot info` command.                                                |
| `pyspigot.command.listscripts`  | Grants permission to use the `/pyspigot listscripts` command.                                         |
| `pyspigot.command.load`         | Grants permission to use the `/pyspigot load` command.                                                |
| `pyspigot.command.loadlibrary`  | Grants permission to use the `/pyspigot loadlibrary` command.                                         |
| `pyspigot.command.reloadall`    | Grants permission to use the `/pyspigot reloadall` command.                                           |
| `pyspigot.command.reload`       | Grants permission to use the `/pyspigot reload` command.                                              |
| `pyspigot.command.reloadconfig` | Grants permission to use the `/pyspigot reloadconfig` command.                                        |
| `pyspigot.command.unload`       | Grants permission to use the `/pyspigot unload` command.                                              |