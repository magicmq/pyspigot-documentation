# Quick Start Guide

Here is a very brief guide for creating your very first PySpigot script. If you don't know Python, that's okay! There are many great online tutorials to learn. See the [Helpful Resources](../scripts/externalresources.md) page for some good Python tutorials.

## Download and Load PySpigot

PySpigot is officially supported on Spigot and Paper and on Minecraft versions 1.16 and newer. I cannot guarantee that PySpigot will work outside of these conditions, but some users report success on other server softwares and/or older MC versions.

PySpigot is built with Java 17 as of version 0.6.0. This means that for version 0.6.0 and later, PySpigot requires Java 17 or above.

Download the latest version of PySpigot from [GitHub](https://github.com/magicmq/pyspigot) or from [Spigot](https://www.spigotmc.org/resources/pyspigot.111006/). Drop the downloaded Jar file into your plugins folder and start your server.

## Creating Your First Script

In this brief tutorial, we will create a very simple script that broadcasts a message to online players.

### Create Script File

All scripts are located in the `scripts` folder within PySpigot's main plugin folder. PySpigot allows for creation of subfolders within the scripts folder for organizational purposes, but script names must be unique across all subfolders.

Create a Python script file here and name it whatever you would like. The file name serves as the name for that script, which will be used to load and unload the script later.

???+ tip

    PySpigot will only load and run script files that end in `.py`. You can easily disable a script without deleting it by changing the file extension (for example, by adding `.disabled` to the end of the file).

### Write The Script

Using the text editor of your choice, open the script file you just created, and write some code:

``` py linenums="1"
from org.bukkit import Bukkit # (1)!

Bukkit.broadcastMessage('Hello world!') # (2)!
```

1.  Here, we import the [`Bukkit`](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/Bukkit.html) class from the Bukkit/Spigot API so that we can reference (use) it later in our script.
2.  Here, we use a function from the `Bukkit` class called `broadcastMessage`. This function broadcasts a message (string) to all players on the server in the server chat.

Save the file, and start your server.

### Run the Script

If you did everything correctly, your script should automatically load on server start. This is expected; PySpigot will automatically load and run all scripts in the `scripts` folder when the plugin loads, including any scripts within subfolders.

Alternatively, if the server is already running and the PySpigot plugin is loaded and enabled, you can load and run your script with `/pyspigot load <scriptname>`. Make sure the name includes the extension (.py)! If the script is located in a subfolder, you don't need to specify the entire path. You only need to specify the script file name.

If you did everything correctly, you should see the message "Hello world!" in the chat when the script loads.

## Next Steps

Check out the rest of the documentation for more advanced scripting.

If you find yourself stuck and need help, [join PySpigot's Discord server](https://discord.gg/f2u7nzRwuk) if you haven't already.