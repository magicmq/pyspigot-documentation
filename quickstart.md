# Quick Start Guide

Here is a very brief guide for creating your very first PySpigot script. If you don't know Python, that's okay! There are many great online tutorials to learn. See the [Helpful Resources](externalresources.md) page for some good Python tutorials.

## Download and Load PySpigot

Download the latest version of PySpigot from GitHub or from Spigot. Drop the downloaded Jar file into your plugins folder and start your server.

## Creating Your First Script

In this brief tutorial, we will create a very simple script that broadcasts a message to online players.

### Create Script File

All scripts are located in the `scripts` folder within PySpigot's main plugin folder.

Create a Python script file here and name it whatever you would like. The file name serves as the name for that script, which will be used to load and unload the script later.

!> PySpigot will only load and run script files that end in `.py`. You can easily disable a script without deleting it by changing the file extension (for example, by adding `.disabled`) to the end of the file.

### Write The Script

Using the text editor of your choice, open the script file you just created, and write some code:

```python
from org.bukkit import Bukkit

Bukkit.broadcastMessage('Hello world!')
```

On line 1, we import the Bukkit class from the Bukkit Java API. This is a very useful class that will likely be used in many of your scripts.

On line 3, we call the `broadcastMessage` method/function, which broadcasts a message to all online players.

### Run the Script

Save the file, and start your server. If you did everything correctly, your script should automatically load on server start. This is expected; PySpigot will automatically load and run all scripts in the `scripts` folder when the plugin loads.

Alternatively, if the server is already running and the PySpigot plugin is loaded and enabled, you can load and run your script with `/pyspigot load <name>`. Make sure the name includes the extension (.py)!

## Next Steps

Check out the rest of the documentation for more advanced scripting.

If you find yourself stuck and need help, join PySpigot's Discord server if you haven't already: https://discord.gg/f2u7nzRwuk