# Quick Start Guide

Here is a very brief guide for creating your very first PySpigot script. If you don't know Python, that's okay! There are many great online tutorials to learn. See the [Helpful Resources](../scripts/externalresources.md) page for some recommended Python tutorials.

## Download and Load PySpigot

Note the official supported versions for your platform:

- **Bukkit:** PySpigot is officially supported on Spigot and Paper and on Minecraft versions 1.16 and newer.
- **Velocity:** PySpigot is officially supported on Velocity version 3.0.0 and newer.
- **BungeeCord:** PySpigot is officially supported on BungeeCord versions corresponding to Minecraft versions 1.16 and newer.

I cannot guarantee that PySpigot will work outside of these conditions, but some users report success on other server softwares and/or older versions. Also note that more recent versions of PySpigot work only on newer versions of Java:

- Prior to version 0.6.0, **Java 12+** is required.
- From version 0.6.0 to version 0.9.0, **Java 17+** is required.
- From version 0.9.1 onwards, **Java 21+** is required.

Download the latest version of PySpigot for your platform from [GitHub](https://github.com/magicmq/pyspigot) or from [Spigot](https://www.spigotmc.org/resources/pyspigot.111006/). Drop the downloaded Jar file into your plugins folder and start your server.

## Creating Your First Script

In this brief tutorial, we will create a very simple script that broadcasts a message to online players.

### Create Script File

All single-file scripts should be placed in the `scripts` folder within PySpigot's main plugin folder. PySpigot allows for creation of subfolders within the scripts folder for organizational purposes, but script names must be unique across all subfolders and across projects (I.E. a script and a project can't share the same name).

Create a Python script file in the `scripts` folder, and name it whatever you would like. Make sure it ends in `.py`. The file name serves as the name for that script, which will be used to load and unload the script later.

???+ tip

    PySpigot will only load and run script files that end in `.py`. You can easily disable a script without deleting it by changing the file extension (for example, by adding `.disabled` to the end of the file). You can also disable a script in its [script options](../scripts/scriptoptions.md).

### Write The Script

Using the text editor of your choice, open the script file you just created, and write some code:

=== "Bukkit"

    ``` py linenums="1"
    from org.bukkit import Bukkit # (1)!

    Bukkit.broadcastMessage('Hello world!') # (2)!
    ```

    1.  Here, we import the [`Bukkit`](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/Bukkit.html) class from the Bukkit/Spigot API so that we can reference (use) it later in our script.
    2.  Here, we use a function from the `Bukkit` class called `broadcastMessage`. This function broadcasts a message (string) to all players on the server in the server chat.

=== "Velocity"

    ``` py linenums="1"
    from dev.magicmq.pyspigot.velocity import PyVelocity # (1)!
    from net.kyori.adventure.text import Component #(2)!

    proxy = PyVelocity.get().getProxy() #(3)!
    message = Component.text('Hello world!') #(4)!

    proxy.sendMessage(message) # (5)!
    ```

    1.  Here, we import the base/core class of PySpigot on the Velocity platform, [`PyVelocity`](https://javadocs.magicmq.dev/pyspigot/dev/magicmq/pyspigot/velocity/PyVelocity.html).
    2.  Here, we import the [`Component`](https://jd.advntr.dev/api/4.9.0/net/kyori/adventure/text/Component.html) class from the Adventure API, to help construct a message.
    3.  Here, we get the Velocity proxy, which will be used later to send the message.
    4.  Here, we construct a new component-based message by using the `Component.text` function from the `Component` class in the Adventure API.
    5.  Here, we use a function from the Velocity proxy called `sendMessage`. This function broadcasts a message (as a Adventure API component) to all players on the proxy.

=== "BungeeCord"

    ``` py linenums="1"
    from net.md_5.bungee.api import ProxyServer # (1)!

    proxy = ProxyServer.getInstance() # (2)!

    proxy.broadcast('Hello world!') # (3)!
    ```

    1.  Here, we import the `ProxyServer` class from the BungeeCord API so that we can reference (use) it later in our script.
    2.  Here, we get the active instance of the `ProxyServer` class (the technicalities of this are beyond the scope of this tutorial).
    3.  Here, we use a function from the `ProxyServer` class called `broadcast`. This function broadcasts a message (string) to all players on the server in the server chat.

Save the file, and start your server.

### Run the Script

If you did everything correctly, the script should load automatically on server start. This is normal behavior- PySpigot will automatically load and run all scripts in the `scripts` folder when the plugin loads, including any scripts within subfolders.

Alternatively, if the server is already running and the PySpigot plugin is loaded and enabled on your platform, you can load and run the script with the load command:

- On Bukkit: `/pyspigot load <scriptname>`
- On Velocity: `/pyvelocity load <scriptname>`
- On BungeeCord: `/pybungee load <scriptname>`

Make sure the name includes the extension (`.py`)! If the script is located in a subfolder, you don't need to specify the entire path. You only need to specify the script file name.

If you did everything correctly, you should see the message "Hello world!" in the chat when the script loads.

## Next Steps

Check out the rest of the documentation for more advanced scripting.

If you're stuck and need help, [join PySpigot's Discord server](https://discord.gg/f2u7nzRwuk).