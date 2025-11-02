# Quick Start Guide for Projects

Here is a very brief guide for creating your very first PySpigot project.

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

In this brief tutorial, we will create a very simple project that registers an event listener.

### Create The Project Folder

All projects should be placed in the `projects` folder of the PySpigot main plugin folder. Each folder within this main `projects` folder is treated as a separate project. All projects must have unique names among other projects and single-file scripts (I.E. a script and a project can't share the same name).

Create a folder within the `projects` folder, and name it whatever you would like. The folder name serves as the name for that project, which will be used to load and unload the project later.

???+ tip

    PySpigot will attempt to load all folders within the `projects` folder (as separate projects). You can disable a project by setting `enabled: false` in its [project options](../projects/projectoptions.md).

### Create a `project.yml`

While not required, it's strongly recommended to create a `project.yml` file inside the root folder of the project. This file serves as the source of [project options](../projects/projectoptions.md) for the project, where project-specific options are placed. PySpigot requires that the file be named `project.yml` and that it be in the root directory of the project.

Create a file called `project.yml` and place it in the folder you created in the previous step.

Using the text editor of your choice, open the file you just created, and add the following:

``` yml linenums="1"
main: 'main.py' # (1)!
```

1. This directive tells PySpigot which file in our project is the "main" module, so it knows which module to execute when the project is loaded.

### Create The Listener Module

Next, create a Python module in the root folder of the project, and name it `listener.py`.

Using the text editor of your choice, open the `listener.py` module, and add some code:

=== "Bukkit"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from org.bukkit.event.player import AsyncPlayerChatEvent # (2)!

    def chat_event(event): # (3)!
        print(event.getMessage()) # (4)!

    def register_events(): # (5)!
        ps.listener_manager().registerListener(chat_event, AsyncPlayerChatEvent) # (6)!
    ```

    1. First, we import the [pyspigot helper module](../scripts/helpermodule.md).
    2. Next, we import the event we want to listen to, [`AsyncPlayerChatEvent`](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/event/player/AsyncPlayerChatEvent.html) in this case.
    3. Next, we define a function that serves as the actual listener for the event. When the event fires, this function will be called. An `AsyncPlayerChatEvent` object will be passed to the event containing data related to the event that occurred.
    4. On this line, we print to console the chat message that was sent.
    5. Here, we create a function that we can call from the main module of the project to register the event listener.
    6. Here register our event with PySpigot's [listener manager](../managers/core/eventlisteners.md), passing the function we defined earlier, as well as the event we want to listen to.

=== "Velocity"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from com.velocitypowered.api.event.player import PlayerChatEvent # (2)!

    def chat_event(event): # (3)!
        print(event.getMessage()) # (4)!

    def register_events(): # (5)!
        ps.listener_manager().registerListener(chat_event, PlayerChatEvent) # (6)!
    ```

    1. First, we import the [pyspigot helper module](../scripts/helpermodule.md).
    2. Next, we import the event we want to listen to, [`PlayerChatEvent`](https://jd.papermc.io/velocity/3.4.0/com/velocitypowered/api/event/player/PlayerChatEvent.html) in this case.
    3. Next, we define a function that serves as the actual listener for the event. When the event fires, this function will be called. A `PlayerChatEvent` object will be passed to the event containing data related to the event that occurred.
    4. On this line, we print to console the chat message that was sent.
    5. Here, we create a function that we can call from the main module of the project to register the event listener.
    6. Here register our event with PySpigot's [listener manager](../managers/core/eventlisteners.md), passing the function we defined earlier, as well as the event we want to listen to.

=== "BungeeCord"

    ``` py linenums="1"
    import pyspigot as ps # (1)!
    from net.md_5.bungee.api.event import ChatEvent # (2)!

    def chat_event(event): # (3)!
        print(event.getMessage()) # (4)!

    def register_events(): # (5)!
        ps.listener_manager().registerListener(chat_event, ChatEvent) # (6)!
    ```

    1. First, we import the [pyspigot helper module](../scripts/helpermodule.md).
    2. Next, we import the event we want to listen to, [`ChatEvent`](https://javadoc.io/doc/net.md-5/bungeecord-api/latest/net/md_5/bungee/api/event/ChatEvent.html) in this case.
    3. Next, we define a function that serves as the actual listener for the event. When the event fires, this function will be called. A `ChatEvent` object will be passed to the event containing data related to the event that occurred.
    4. On this line, we print to console the chat message that was sent.
    5. Here, we create a function that we can call from the main module of the project to register the event listener.
    6. Here register our event with PySpigot's [listener manager](../managers/core/eventlisteners.md), passing the function we defined earlier, as well as the event we want to listen to.

Save the file and exit.

### Create The Main Module

Lastly, create another Python module in the root folder of the project, and name it `main.py`.

Using the text editor of your choice, open the `main.py` module, and add some code:

=== "Bukkit"

    ``` py linenums="1"
    import listener # (1)!
    from org.bukkit import Bukkit # (2)!

    listener.register_events() # (3)!
    Bukkit.broadcastMessage('Registered event listener!') # (4)!
    ```

    1. First, we import the listener module created in the previous step.
    2. Next, we import the `Bukkit` class to broadcast a message.
    3. Next, we call the function `register_events` in the listener module to register the listener we wrote previously.
    4. On this line, we broadcast a message to the server with the `broadcastMessage` function informing that the listener was registered.

=== "Velocity"

    ``` py linenums="1"
    import listener # (1)!
    from dev.magicmq.pyspigot.velocity import PyVelocity # (2)!
    from net.kyori.adventure.text import Component # (3)!

    proxy = PyVelocity.get().getProxy() # (4)!

    listener.register_events() # (5)!
    proxy.sendMessage(Component.text('Registered event listener!')) # (6)!
    ```

    1. First, we import the listener module created in the previous step.
    2. Next, we import the `PyVelocity` class to get the proxy instance.
    3. Next, we import the `Component` class from the Adventure API to construct a message to broadcast.
    4. Next, we get the proxy instance.
    5. Next, we call the function `register_events` in the listener module to register the listener we wrote previously.
    6. On this line, we broadcast a message to the server with the `broadcastMessage` function informing that the listener was registered.

=== "BungeeCord"

    ``` py linenums="1"
    import listener # (1)!
    from net.md_5.bungee.api import ProxyServer # (2)!

    proxy = ProxyServer.getInstance() # (3)!

    listener.register_events() # (4)!
    proxy.broadcast('Registered event listener!') # (5)!
    ```

    1. First, we import the listener module created in the previous step.
    2. Next, we import the `Bukkit` class to broadcast a message.
    3. Next, we get the proxy instance.
    4. Next, we call the function `register_events` in the listener module to register the listener we wrote previously.
    5. On this line, we broadcast a message to the server with the `broadcastMessage` function informing that the listener was registered.

### Running The Project

If you did everything correctly, the project should automatically load on server start. This is expected; PySpigot will automatically load and run all scripts and projects in the `scripts` and `projects` folders when the plugin loads.

Alternatively, if the server is already running and the PySpigot plugin is loaded and enabled, you can load and run the project with the load command:

- On Bukkit: `/pyspigot load <projectname>`
- On Velocity: `/pyvelocity load <projectname>`
- On BungeeCord: `/pybungee load <projectname>`,

passing the name of the root project folder of the project for the project name.

Verify that the project is loaded and running and has a listener registered for the platform-specific player chat event with the info command:

- On Bukkit: `/pyspigot info <projectname>`
- On Velocity: `/pyvelocity info <projectname>`
- On BungeeCord: `/pybungee info <projectname>`

## Next Steps

Check out the rest of the documentation for more advanced scripting.

If you're stuck and need help, [join PySpigot's Discord server](https://discord.gg/f2u7nzRwuk).