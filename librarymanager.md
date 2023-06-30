# External Libraries

Because PySpigot scripts complile into Java bytecode (and are executed through Java), they have access to the entire Java class path at runtime. This means that a script can access any Java class that is loaded and running on the server.

There are several open source Java libraries that provide various QOL functions that make writing code easier. the [Apache Commons Libraries](https://commons.apache.org/) are one such example.

If you find a library you would like to use with a script but it is not loaded on your server already, you can load the library using PySpigot's library manager.

## Loading Libraries

All Jar files located within the `libs` folder of the PySpigot main plugin folder will be loaded on server start/when the PySpigot plugin is enabled. To load a library at a later time, follow these steps:

1. Drop the libary you would like to use into the `libs` folder, located within PySpigot's main plugin folder.
2. Load the library with the command `/loadlibrary <filename>`, where `<filename>` is the name of the Jar file library you would like to load.
3. Import the library in your script, for example, `from org.apache.commons.text import StringEscapeUtils`

!> PySpigot currently does not have a way to unload previously loaded libraries. This may be added in a future release.