# Autocomplete and Code Suggestions

???+ warning

    Currently, this is an **experimental project**. If you choose to set up and use this portion of PySpigot, expect bugs and frequent changes. I will notify of any updates and changes on Discord, so be sure to join if you haven't already, to stay in the loop.

If you're looking for instructions on how to set this up for yourself, skip down to the [setup section](#setup).

One difficulty you might encounter when writing PySpigot scripts is a lack of code suggestions and autocomplete in your IDE when you write them. The reason why these features aren't available when writing scripts is relatively straightforward - because PySpigot scripts are executed *at runtime*, the IDE/text editor has **no idea** what functions, classes, etc. you are referencing when writing scripts.

Writing more complex scripts can quickly become cumbersome because none of the API is available, so you would need to manually look everything up, including packages, class names, functions, and API documentation. This project is aimed at smoothing out these difficulties when writing scripts.

## Overview

The core of this project is [docs-translator](https://github.com/magicmq/docs-translator), a Java program which, in very simple terms, takes documented Java source code, translates it into documented Python code, and then bundles the translated sources into an installable Python package. These Python packages can then be installed into a virtual environment, which an IDE can use.

### How docs-translator Works

docs-translator relies heavily upon the [JavaParser](https://javaparser.org/) library, which, in the most simple terms, reads Java source (`.java`) files and turns them into an [abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree), which can be read programmatically and translated into Python source files with relative ease.

The application runs in a stepwise fashion:

1. The application initializes all working directories.
2. Java source JAR files (I.E. those that follow the format `*-sources.jar`) are downloaded from remote repositories/URLs.
3. The application loops through all files contained within downloaded source JAR files. When it encounters a Java soruce file (ending in `.java`), the file is parsed with JavaParser and a best effort attempt is made to translate the source file into Python code.
    * Any files not ending in `.java` are ignored.
4. Translated `.py` files are placed in the user-defined output folder (`generated` by default), in the appropriate package.
5. An entry is added to the `__init__.py` file in the appropriate package, to allow for importing the python module as one would normally import a Java class in Jython.
6. Any source files from the Java Standard Library utilized by the previously translated Java source files are also translated in the same process outlined above.
7. All `__init__.py` files are generated and placed in their appropriate locations.
8. Python package-related files (`setup.py`, `pyproject.toml`, `MANIFEST.in`, `LICENSE`) are generated from options specified in the `settings.yml` and are placed in the user-defined output folder (`generated` by default).

I then build the generated python package with `python -m build`, and upload it to PyPI with `twine`. Currently, this step of the process is manual, but I am considering expanding the functionality of docs-translator to automate this step as well.

## Setup

Setup is relatively simple and involves creating a new Python virtual environment, installing the necessary packages from PyPI using `pip`, and then designating the virtual environment as your Python installation in your IDE of choice.

Although this should work with any IDE, this example uses [Visual Studio Code (VS Code)](https://code.visualstudio.com/), as this IDE is popular, free, feature-rich, and it is what I use when writing Python. I also haven't tested how well this works in other IDEs.

### Prerequisites

- Ensure **Python 3** is installed on your system, and that you are running **Python 3.9 or higher**. Although PySpigot only supports Python 2, Python 3 is required for this portion of the project. You can check by running:
    ```
    python --version
    ```
- An IDE of your choice. VS Code is preferred and is used in this guide.
- A project directory where your scripts and projects live. This could be the `PySpigot` plugin folder or a separate directory if you want to keep your development environment separate from your server (or your server is on a different machine).
    - This is also the preferred location for the Python virtual environment.

### Create a Python virtual environment

???+ note

    While not strictly required for setting this up, creating a virtual environment is **strongly recommended** because the Python packages containing the translated Java sources may interfere with other native Python packages in unpredictable ways. Using a virtual environment ensures that the translated sources remain isolated from native Python packages installed globally.

=== "Windows"

    1. Open **Command Prompt (cmd)** or **PowerShell**.
    2. Navigate to your project directory:
        ```
        cd path/to/your/project/directory
        ```
    3. Create a virtual environment with Python:
        ```
        python -m venv venv
        ```
        - Note that in the above command, the last argument is the name of the virtual environment. In this case, we're naming it `venv`.
    4. Activate the virtual environment:
        - **Command Prompt:**
            ```
            venv\Scripts\activate
            ```
        - **PowerShell:**
            ```
            venv\Scripts\activate.ps1
            ```
            *(Note: If using PowerShell, you may need to enable script execution using `Set-ExecutionPolicy Unrestricted -Scope Process`)*
    5. The virtual environment is now active, and you should see `(venv)` in the command line prompt.
    6. To deactivate the virtual environment:
        ```
        deactivate
        ```

=== "macOS & Linux"

    1. Open a **terminal**.
    2. Navigate to your project directory:
        ```
        cd path/to/your/project/directory
        ```
    3. Create a virtual environment with Python:
        ```
        python -m venv venv
        ```
        - Note that in the above command, the last argument is the name of the virtual environment. In this case, we're naming it `venv`.
    4. Activate the virtual environment:
        ```
        source venv/bin/activate
        ```
    5. The virtual environment is now active, and you should see `(venv)` in the command line prompt.
    6. To deactivate the virtual environment:
        ```
        deactivate
        ```

### Install the translated sources

As stated previously, translated sources have been bundled into Python packages and published to PyPI for convenient installation. At minimum, **two** packages should be installed into the virtual environment:

- **The translated sources of the server software you are developing for:**

    | If your server runs... | Then the package you should install is... |
    | ---------------------- | ----------------------------------------- |
    | Spigot                 | pyspigot-spigot-sources                   |
    | Paper                  | pyspigot-paper-sources                    |
    | Purpur                 | pyspigot-purpur-sources                   |
    | Mohist                 | pyspigot-spigot-sources                   |
    | BungeeCord             | pyspigot-bungee-sources                   |

    The version to install is the Minecraft version of your server.

    Some examples:

    - If your server runs Spigot on Minecraft 1.18.2, install `pyspigot-spigot-sources==1.18.2`.
    - If your server runs Paper on Minecraft 1.21.4, install `pyspigot-paper-sources==1.21.4`.
    - If your server runs Purpur on Minecraft 1.20.6, install `pyspigot-purpur-sources==1.20.6`.
    - If your server runs BungeeCord on Minecraft 1.19.4, install `pyspigot-bungeecord-sources==1.19.4`.

- **The translated sources of PySpigot itself:**

    Determining which PyPI package to install for the translated sources of PySpigot is much easier:

    | If your server runs...               | Then the package you should install is... |
    | ------------------------------------ | ----------------------------------------- |
    | Bukkit (Spigot, Paper, Purpur, etc.) | pyspigot-sources-bukkit                   |
    | BungeeCord                           | pyspigot-sources-bungee                   |

    The version to install is the version of PySpigot on your server.

    Some examples:

    - If your server runs Bukkit and PySpigot version 0.9.0, install `pyspigot-sources-bukkit==0.9.0`.
    - If your server runs BungeeCord and PySpigot version 0.9.0, install `pyspigot-sources-bungee==0.9.0`.

**After you've figured out what packages you need to install, follow these steps:**

1. Ensure the virtual environment you created earlier is activated:
    - Verify you see `(venv)` in the command line prompt.
    - Run the following:
        ```
        where python (Windows)
        which python (macOS/Linux)
        ```
        Verify the output includes the location where you just installed the virtual environment.
2. Install each package:
    ```
    pip install pyspigot-spigot-sources==1.21.4
    pip install pyspigot-sources-bukkit==0.9.0
    ```
    *Note: These will differ depending on your server software, Minecraft version, and PySpigot version! See the above section if you aren't sure what to install.*
3. Verify all packages were installed correctly:
    ```
    pip freeze
    ```
    Each package you installed should be listed.

### Configure VS Code to use the virtual environment

1. Open **VS Code** and navigate to/open your project directory.
    - **Windows:** If you're already in the project directory from the previous step, you can type the following into Command Prompt/PowerShell to open VS Code:
    ```
    code .
    ```
2. Open the **Command Palette**:
    - **Windows/Linux:** <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd>
    - **macOS:** <kbd>:material-apple-keyboard-command:</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd>
3. Search for and select **Python: Select Interpreter**.
4. In the list that appears, select the interpreter located inside the virtual environment you created earlier:
    - **Windows**: ./venv/Scripts/python.exe
    - **macOS/Linux**: ./venv/bin/python
5. Ensure the selected interpreter appears in the **bottom-right corner** of VS Code.
6. VS Code will now automatically activate the virtual environment

???+ note

    If you write scripts for multiple server softwares and/or Minecraft versions, **you must create a different virtual environment for each server software and Minecraft version**. Installing translated sources for multiple different server softwares/Minecraft versions into the same virtual environment *will* result in conflicts and overwrites.

That's it! Now, when you write scripts, VS Code should now give code suggestions, autocomplete suggestions, and show API documentation, since it can now pull this information from the translated sources that were installed into the virtual environment.

## Using Type Hints When Writing Scripts

Type hints are useful to your IDE so that it can provide code suggestions and autocomplete in situations where types are ambiguous. These situations arise when defining functions for event listeners, commands, tasks, and other situations where PySpigot calls functions in your script from Java. For example, consider the following script that registers an event listener:

``` py linenums="1"
import pyspigot as ps
from org.bukkit.event.player import AsyncPlayerChatEvent

def player_chat(event): # (1)!
    print('Player sent a chat! Their message was: ' + event.getMessage())

listener = ps.listener.registerListener(player_chat, AsyncPlayerChatEvent)
```

1. Your IDE will have no idea what type `event` is, so it  won't be able to provide any code suggestions or autocomplete for this variable.

Your IDE won't be able to provide code suggestions when calling `event.<something>` from within the `player_chat` function, because it doesn't know the type of `event`. We can add a type hint to the `event` parameter to remedy this ambiguity:

``` py linenums="1"
import pyspigot as ps
from org.bukkit.event.player import AsyncPlayerChatEvent

def player_chat(event: "AsyncPlayerChatEvent"): # (1)!
    print('Player sent a chat! Their message was: ' + event.getMessage())

listener = ps.listener.registerListener(player_chat, AsyncPlayerChatEvent)
```

1. Now, the IDE knows that `event` will be of type `AsyncPlayerChatEvent`, so it can now provide code suggestions based on the functions within the `AsyncPlayerChatEvent` class.

???+ note

    Since type hints were introduced in Python 3.5, Jython won't be able to parse them. You must remove them manually before running your script. I plan to add a feature into PySpigot that removes type hints automatically when loading a script file, to make things easier on the user end.

## Installing Additional Libraries

The beauty of this system is that sources of **any** Java library can be translated and installed into your virtual environment for use when writing scripts. If there is an external library you use that you would like me to translate and publish to PyPI, feel free to reach out to me on Discord.

## Issues

Since this is a new project, and I expect issues. If you have any, please report them either on Discord or on the [docs-translator Github page](https://github.com/magicmq/docs-translator/issues).